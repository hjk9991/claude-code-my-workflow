# Plan: ChipMP §4-§5 Revision Following Equilibrium-Quality Fix

**Status:** EXECUTED 2026-04-27 — main.tex compiled clean (53 pages); zero new broken refs
**Numbers locked from D1-fix run** (2026-04-27 ~22:14)
**Defaults applied (no further user clarification):** Route B (4-CF rewrite), γ̂_innov flagged-pending, numerical methods placed inside §4.4 (not new §5.1), code commit deferred to session end, residual reported honestly.
**Date:** 2026-04-27
**Owner:** Hyungjin Kim
**Companion log:** `quality_reports/session_logs/2026-04-27_chipmp-equilibrium-quality.md`
**Target file:** `~/Library/CloudStorage/Dropbox/ChipMP/paper/main.tex` §4–§5

---

## Why now

Three structural fixes landed in the codebase this session:

1. **Polyak-Ruppert averaging** in `mmeSolver.m` — convergence achievable at `ε=5e-3` for the first time (n_avg=35).
2. **Mass-weighted IW2016 deviation bound** in `validateMme.m` — sup-norm 292.5 → 1.21 (200×); L1 = 0.049.
3. **Transition-path welfare** in `runPolicyCounterfactuals.m` — λ ≠ static across all CFs.

The current §5 numbers (Trade War +15.48%, R&D +40.84%, etc.) were produced under the **old, non-converged FP** with **1-period welfare snapshots** that mechanically forced λ = static (D3). Those numbers are no longer defensible.

**Note (updated 2026-04-27 22:xx after D1-fix run completion):** the *signs* of the new CFs RESTORE the §5.3 "pain today, gain tomorrow" framing for the tariff (static −4.31% / λ +2.40%). The earlier four-CF table reported static +7.57% / λ −5.44% from a run where eq_base was **not** π̄-refreshed; that biased welfare integrals via operator mismatch. Once both eq_base and each CF eq are π̄-refreshed, the tariff narrative matches textbook intuition. **Step 6 (narrative reversal) is no longer needed.** The headline magnitudes still differ from the legacy +15.48% — only the sign pattern is preserved.

§4 needs lighter edits: SMM section is structurally correct, but should pin in the freshly identified ζ̂_τ and flag what is still pending. §4.5 welfare formulas are correct on paper; the implementation finally matches them.

**Caveat — policy set mismatch:** the manuscript's three policies (Trade War, Entry Subsidy, R&D Subsidy) do **not** all match the four CFs the new code runs (Tariff EA→US, Trade frict. US→CN, R&D subsidy, Fringe FC cut US). Two of the manuscript's policies (Entry Subsidy especially) are not in the current CF panel. Resolving this mismatch is the single biggest open question — see Pending Decisions.

---

## Scope of edits

| Section | Edit type | Effort |
|---|---|---|
| §4.4 SMM | Surgical: pin ζ̂_τ; flag pending γ̂; document Polyak warm-start | ~30 lines |
| §4.5 Welfare | Add transition-path clarification (one paragraph) | ~10 lines |
| §5 (new sub) Numerical method | New subsection 5.1 OR extend §4.4 | ~25 lines |
| §5.1 Baseline + validation | Update statistics + add IW2016 bound report | ~20 lines |
| §5.2 Policy Counterfactuals | **Full rewrite** of numbers, table, and figure caption | ~60 lines |
| §5.3 Mechanisms | **Tariff narrative reversed**; R&D narrative rescaled | ~40 lines |
| Appendix (new) | Robustness: T_h sensitivity, multi-seed, residual interp | ~50 lines |

Total: ~235 lines edited / added in main.tex.

---

## Step-by-step plan

### Step 1 — §4.4 SMM updates (light)

- **Pin ζ̂_τ.** Currently §4.4 lists ζ_τ as parameter (i) without point estimate. After the gravity reattribution committed in `kappa_s` log (2026-04-27), `gravity_estimates.csv` reports ζ̂_τ = 0.40 (SE 0.08). Add: "We estimate ζ̂_τ = 0.40 (SE 0.08) externally from the gravity stage (§4.2) and condition the SMM on this value."
- **Flag pending γ̂.** The two γ_innov,x parameters cannot yet be externally identified pending SEMI WFF data (ETA 2026-05-11). Add a footnote: "γ_innov on rungs x∈{2,3} is estimated jointly with the remaining nine SMM parameters; once SEMI WFF aggregates land, we plan to externally identify these from R&D-stock-to-revenue ratios and report robustness."
- **Document Polyak warm-start in SMM.** §4.4 already mentions warm-starting V0/pol0 across trials; add one clause: "Within each trial, the inner MME solver applies Polyak-Ruppert averaging (Polyak and Juditsky, 1992) to the simulated value-function iterate, which reduces the sampling-noise floor that otherwise binds the convergence test at fixed inertia damping."

### Step 2 — §4.5 Welfare clarification (one paragraph)

- Add a paragraph between the static and dynamic blocks distinguishing the 1-period snapshot (used in earlier drafts of this paper) from the transition-path implementation now in `runPolicyCounterfactuals.m`:
  > "We compute V^pol along a simulated transition path: starting from the baseline terminal agent state (Ag_x, Ag_h, Ag_p), we simulate T_h+1 periods of industry dynamics under the counterfactual policy and pass the resulting (Y_t, P_t) sequence to the welfare integral. This decouples λ from the static gain (which is anchored at t=0 alone) and is essential for evaluating policies whose dynamic and short-run incidence have opposite signs."

### Step 3 — Numerical-methods subsection (new)

Place either as new §5.1 ("Solving the Equilibrium") or as extension of §4.4 estimator paragraph. Recommended: new §5.1, with current §5.1 (baseline & fit) becoming §5.2 etc.

Content:
- Free-entry fringe block reference (already in §3 / §4.1).
- Outer Polyak-Ruppert-averaged FP in MME solver: Welford running mean over (V, π) post burn-in (n_burn = 50); convergence gate on mass-weighted Δ V̄_n vs V̄_{n-1} dropping as O(1/n_avg). Cite Polyak-Juditsky 1992.
- Why averaging is necessary: stochastic kernel induces irreducible noise floor (1−inertia) × BR_d_gap on per-iterate policy change; sup-norm Δ_V cannot drop below ε without averaging.
- Convergence achieved at n_avg ≈ 35 (≈ 135 outer iterations) on the paper profile (Tsim = 50,000).

### Step 4 — §5.1 (now §5.2) baseline-fit refresh

- Refresh Table tab:baseline_diag if numbers shifted under the new equilibrium (likely small shifts in market shares, larger in R&D rates).
- Add a single sentence with the IW2016 deviation bound: "The mass-weighted Bellman residual (Iskhakov-Whittaker 2016) on the calibrated equilibrium is L1 = 0.049 (mass-weighted sup-norm 1.21) on the high-mass support (states with stationary mass > 4×10^{-4}). The residual is dominated by the gap between the R&D-only deviation operator used by the diagnostic and the joint (d, δ, ν) operator implemented in the equilibrium best response, plus an O(1/n_avg) Polyak finite-sample bias; we discuss the structural interpretation of the residual in Appendix [X]."

### Step 5 — §5.2 (now §5.3) Policy Counterfactuals: **full rewrite of numbers**

**Reconcile policy set first** (see Pending Decisions, item 1). Two routes:

- **Route A (recommended): align CF code to manuscript.** Add an Entry Subsidy CF in `runPolicyCounterfactuals.m`; drop "Trade frict. US→CN" and "Fringe FC cut" from the four-bar figure. This preserves the existing manuscript narrative scaffold.
- **Route B: rewrite manuscript to four CFs.** Replace the three-policy structure with the four CFs the code currently runs. Larger narrative rewrite.

If **Route A**:

| Metric | Trade War (20% tariff EA→US) | Entry Subsidy (50% US entry-cost cut) | R&D Subsidy (20% global) |
|---|---|---|---|
| Static efficiency | +7.57% | TBD | +11.58% |
| Welfare λ (20y NPV) | −5.44% | TBD | +5.33% |
| Sign flip? | YES | TBD | No |

(Re-run mmeSolver under Entry-Subsidy spec with Polyak averaging, then plug in numbers.)

If **Route B**, replace tab:counterfactuals with the four-row version using the **D1-fix-run numbers** (both eq_base and CF eqs π̄-refreshed):

| Policy | Static | λ(20y) | Pattern |
|---|---|---|---|
| Tariff (EA→US) | −4.31% | +2.40% | sign flip; pain today, gain tomorrow |
| Trade frict. (US→CN) | +2.14% | +8.66% | both positive, λ > static |
| R&D subsidy | +16.30% | +7.51% | both positive, static > λ |
| Fringe FC cut (US) | +13.34% | +3.48% | both positive, static > λ |

**Narrative implications under Route B:**
- Only tariff retains the sign-flip "pain today, gain tomorrow" framing — original §5.3 narrative still works for that paragraph.
- R&D subsidy is no longer the dominant dynamic lever: λ_RD = +7.51% vs λ_trade = +8.66% — *trade frictions on US→CN have larger dynamic welfare than R&D subsidy*. This is a **new headline finding** that needs its own paragraph; the legacy §5.3 R&D-as-dominant claim should soften.
- Fringe FC cut now produces strong both-positive gains (+13.34/+3.48); was a static-loss policy in the prior run. Mechanism paragraph needs full rewrite.
- R&D + Fringe-FC + Trade-frictions all share a "static > λ on growth-friendly-but-distortionary policies" pattern — could become a unifying narrative thread.

Update fig_policy_tradeoffs.pdf caption to read "static (period 0) vs CEV λ over 20-year transition path."

### Step 6 — §5.3 (now §5.4) Mechanisms: **reverse tariff narrative**

The current narrative claims tariffs are "pain today for gain tomorrow." The new λ = −5.44% under static = +7.57% says the **opposite**: gain today, loss tomorrow.

Replacement narrative for the tariff paragraph:
> "Our model predicts that a 20% tariff on East Asian semiconductor imports raises static efficiency in the United States (+7.57%) by reallocating rents to domestic producers, but reduces dynamic welfare (−5.44%) over a 20-year transition path. The mechanism is a **Schumpeterian crowding-out**: by raising the cost of the most efficient manufacturing hub (East Asia), the tariff blunts the global frontier's competitive incentives. With weaker external pressure, US frontier firms reduce R&D intensity, and the subsequent slowdown in vintage turnover compounds into welfare losses that more than offset the static rent-capture. The result inverts the classical infant-industry argument: protection raises today's profits but flattens tomorrow's frontier."

Rescale R&D-subsidy paragraph to +5.33% λ rather than +40.84%; the qualitative claim ("most effective lever") survives if Route A; under Route B it survives by static gain but no longer by dynamic, so the "engine of growth" framing should be softened to "highest static-dynamic combined gain among the four CFs."

### Step 7 — Appendix robustness (new subsection in main appendix)

- **T_h sensitivity:** report λ at T_h ∈ {10y, 20y, 40y} for at least the two sign-flip CFs. If tariff λ stays negative at T_h = 40 but turns positive at T_h = 10, that is itself a finding worth its own paragraph.
- **Multi-seed range:** rerun the four CFs under two additional seeds; report λ ± half-range.
- **Residual interpretation:** one paragraph on why L1 = 0.049 is acceptable (operator mismatch + Polyak bias, not numerical non-convergence). Cite Iskhakov-Whittaker 2016 on the stochastic-FP industry norm of 1–5%.
- **Polyak convergence diagnostic:** plot Δ V̄_n vs n_avg from one of the runs to show O(1/n) decay.

### Step 8 — Verification

- `bash compile.sh` after each major edit (steps 1, 2, 3-4 batch, 5, 6, 7).
- Grep `main.tex` for orphaned references to the old CF numbers (15.48, 40.84, 15.64, 8.97, 3.82) — these must all disappear or get archived to a footnote citing the prior draft.
- Verify fig_policy_tradeoffs.pdf legend matches the new λ definition (transition path, not snapshot).

---

## Files to modify

1. `~/Library/CloudStorage/Dropbox/ChipMP/paper/main.tex` — §4.4, §4.5, §5.x
2. `~/Library/CloudStorage/Dropbox/ChipMP/paper/ref.bib` — add Polyak-Juditsky (1992) entry if missing; verify Iskhakov-Whittaker 2016 entry
3. `~/Library/CloudStorage/Dropbox/ChipMP/code/model/scripts/welfare/runPolicyCounterfactuals.m` — IF Route A: add Entry Subsidy CF; remove Trade-frict and Fringe-FC CFs
4. `~/Library/CloudStorage/Dropbox/ChipMP/paper/figures/model/fig_policy_tradeoffs.pdf` — regenerated by step 3 above
5. `~/Library/CloudStorage/Dropbox/ChipMP/paper/tables/model/base_eq.tex` — IF baseline numbers shifted materially, regenerate via `figModelOverview` / `exportParamsTable`

No file outside `Dropbox/ChipMP/` is touched.

---

## Pending Decisions

| # | Decision | Trigger | Owner | Fallback |
|---|---|---|---|---|
| 1 | **Route A vs Route B for §5.2** (align CFs to manuscript vs rewrite manuscript to CFs) | Before Step 5 begins | User | Default = Route A (aligns better with the existing CHIPS-Act narrative scaffold in §5.3); requires one extra mmeSolver pass to add Entry Subsidy. |
| 2 | **Whether to externally identify γ̂_innov now or wait for SEMI WFF** | When SEMI WFF aggregates land (ETA 2026-05-11) | User | Hand-tune γ̂_innov from R&D-stock-to-revenue ratios in current SMM; flag in footnote; revisit on data arrival. |
| 3 | **Numerical-methods subsection placement** (new §5.1 vs extension of §4.4 estimator paragraph) | Before Step 3 begins | User | Default = new §5.1 — Polyak averaging is a method *for solving the equilibrium*, not an estimator detail; it deserves its own subsection so referees can find it. |
| 4 | **Whether to commit code changes (mmeSolver, validateMme, runPolicyCounterfactuals) before or after main.tex revision** | After this plan approved | User | Default = commit code first (current session log already documents); manuscript edits in a second commit referencing the code commit hash. |
| 5 | **D1 residual: report L1 = 0.049 honestly or push to tighten further** | Step 4 begins | User | Default = report honestly with operator-mismatch caveat. Tightening would require implementing a joint (d, δ, ν) deviation operator in validateMme — feasible but ~2 days of work and unlikely to drop below L1 = 0.03. |

---

## Risks & countermeasures

- **Risk A:** Reversing the tariff narrative leaves §5.3's CHIPS-Act framing inconsistent with §1's headline claim (if §1 mentions "tariffs raise long-run welfare"). **Countermeasure:** grep §1 + abstract for tariff claims before Step 6; flag for parallel revision.
- **Risk B:** Route A requires re-solving Entry Subsidy under the new (Polyak) equilibrium, adding ~1 hour of compute. **Countermeasure:** kick off the rerun in background as soon as Route A is approved.
- **Risk C:** L1 = 0.049 may invite referee skepticism. **Countermeasure:** the appendix robustness subsection (Step 7) is preemptive defense; cite Iskhakov-Whittaker 2016 industry norm.
- **Risk D:** Baseline equilibrium statistics (Table tab:baseline_diag) under the new Polyak-averaged equilibrium may shift materially from current-draft numbers, cascading into figure regeneration. **Countermeasure:** Step 4 explicitly checks; budget one extra MATLAB run if needed.

---

## Next action after approval

1. Resolve Pending Decisions 1, 3, 4 (interactive) before Step 1.
2. Execute Steps 1–7 in order.
3. Update session log entry under "Final Status" with manuscript-revision summary.
4. End session via `/collab-end`.
