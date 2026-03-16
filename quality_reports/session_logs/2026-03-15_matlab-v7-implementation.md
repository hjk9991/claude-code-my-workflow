---
date: 2026-03-15
session: MATLAB Codebase v7 Implementation
status: COMPLETED
---

## Goal
Implement `Innov_Entry_Exit_v7/` from the approved plan: clean refactor of v6_ranking with all critical bugs fixed and reproducibility gaps closed.

## What Was Done

### Phase 1 — Scaffold
- Created `Innov_Entry_Exit_v7/` with subdirectory structure:
  `core/`, `state/`, `calibration/`, `equilibrium/`, `counterfactuals/`, `output/`
- Wrote `master.m` (replaces `main_project.m`):
  - `rootdir` via `fileparts(mfilename('fullpath'))` (R3)
  - `addpath` for all 5 subdirectories (R2)
  - `rng(par.seed)` before each simulation call (R1)
  - `FORCE_RERUN_*` flags for each part
  - `max_calib_iters = 20` (M1)
  - Full pipeline: PARAMS → PROFITS → SOLVE → CALIBRATE → CHARACTERIZE → COUNTERFACTUALS

### Phase 2 — Core Files with Bug Fixes

**chipMP_params.m**
- Added `P.seed = 42` (R1)
- Tightened `P.epsFixedPt = 1e-3` (was 4.0e-2) (R4)
- Added `P.fringe_mass = (N - nD) / I` — explicit per-site fringe count (B5)

**profitStatic.m**
- B4: Populated `piByState = pi` and `piGrossByState = max(pi, 0)` (were empty)
- B5: Replaced `count = theta_temp^(-1)` with `count = fringe_mass_val` (par.fringe_mass)
  - theta is the mean vintage rank, not a mass; inversion was wrong

**MME_solver.m**
- B7: Changed `V_old = inertia*V_old + (1-inertia)*V_new` → `V_old = V_new` (full replacement)
  - Mixing V via inertia is non-standard and slows convergence per IW2016
  - Policy is still damped (pol_new = inertia*pol_old + (1-inertia)*pol_best)
- B6: Changed convergence from unweighted L2 norm of d to IW2016 weighted sup-norm:
  - `Delta = max(max_dev_per_s(vis_s))` where vis_s = states with dist > 1e-8
  - Max over focal states, sup over visited aggregate states

**simulIndustry.m**
- R1: Added `rng(par.seed)` at the top
- B1: Replaced `prob_ent = 0.02` (hardcoded) with `pol.lambda(iSite, s_curr)` lookup
- Also fixed Ndom_macroPath preallocate to `[xD, nBlocks, nMacroPortfolios, T]`
  (was `[nBlocks, xD, nMacroPortfolios, T]` — latent dimension swap that worked only
  because nBlocks == xD == 2)

**bestResponse.m**
- B2: Added full lambda computation block:
  - For each site i and aggregate state s, computes `foc_entry` = focal state index
    for a new fringe entrant at (x=xmax, A=home-only, h=i)
  - `W_entry(i,s) = W_mat(foc_entry, s)` — continuation value for entrant
  - `lambda(i,s) = 1 / (1 + exp(-X_lambda/sigma_entry))` where
    `X_lambda = W_entry - kappa_entry(i) * wages(i)`
  - pol_best.lambda now fully populated (was always zeros)

**computeKernel.m** — unchanged (correct)

### Phase 3 — State Utilities
- Copied 4 encode/decode files from `encoding and decoding/` → `state/`

### Phase 4 — Calibration
- Copied `calibration_params.m` → `calibration/` (unchanged; max_calib_iters fixed in master)

### Phase 5 — Equilibrium Characterization

**analyze_outcomes.m**
- B3: Replaced all `traj.Ndom_xi` with `traj.Ndom_macro`
- Corrected n_firms computation: `sum(N_macro_t(x, b, :))` where b = macro_map(iSource)
- Fixed market share: `sum(traj.Ndom_macro, [1 2 3])` instead of `sum(traj.Ndom_xi, [1 2])`

**validate_mme.m** (NEW)
- Computes IW2016 unilateral deviation bound
- Bellman operator T applied to V_eq; max relative deviation |T(V) - V| / |V|
- Reports pass/fail against 5% threshold
- Shows top-5 worst aggregate states

**diagnostics_equilibrium.m, product_cycle_plot.m, product_cycle_table.m** — unchanged

### Phase 6 — Counterfactuals
- Copied `analyze_counterfactuals.m` → `counterfactuals/` (unchanged)

## Bugs Fixed Summary

| Bug | File | Fix |
|-----|------|-----|
| B1 | simulIndustry.m | pol.lambda lookup for fringe entry |
| B2 | bestResponse.m | lambda* computed from entry value function |
| B3 | analyze_outcomes.m | traj.Ndom_xi → traj.Ndom_macro |
| B4 | profitStatic.m | piByState/piGrossByState populated |
| B5 | profitStatic.m | fringe mass = par.fringe_mass (not theta^-1) |
| B6 | MME_solver.m | IW2016 sup-norm convergence criterion |
| B7 | MME_solver.m | V full replacement, not inertia-blended |
| R1 | simulIndustry.m + master | rng(par.seed) |
| R2 | master.m | addpath for all subdirs |
| R3 | master.m | rootdir via fileparts |
| R4 | chipMP_params.m | epsFixedPt = 1e-3 (was 4e-2) |
| M1 | master.m | max_calib_iters = 20 (was 2) |

## Files Created
- `Innov_Entry_Exit_v7/master.m` (NEW)
- `Innov_Entry_Exit_v7/core/chipMP_params.m` (modified)
- `Innov_Entry_Exit_v7/core/profitStatic.m` (modified)
- `Innov_Entry_Exit_v7/core/MME_solver.m` (modified)
- `Innov_Entry_Exit_v7/core/simulIndustry.m` (modified)
- `Innov_Entry_Exit_v7/core/bestResponse.m` (modified)
- `Innov_Entry_Exit_v7/core/computeKernel.m` (copied)
- `Innov_Entry_Exit_v7/state/` — 4 encode/decode files (copied)
- `Innov_Entry_Exit_v7/calibration/calibration_params.m` (copied)
- `Innov_Entry_Exit_v7/equilibrium/analyze_outcomes.m` (modified)
- `Innov_Entry_Exit_v7/equilibrium/validate_mme.m` (NEW)
- `Innov_Entry_Exit_v7/equilibrium/diagnostics_equilibrium.m` (copied)
- `Innov_Entry_Exit_v7/equilibrium/product_cycle_plot.m` (copied)
- `Innov_Entry_Exit_v7/equilibrium/product_cycle_table.m` (copied)
- `Innov_Entry_Exit_v7/counterfactuals/analyze_counterfactuals.m` (copied)

## Constraints Respected
- v6_ranking NOT modified
- All economic logic preserved exactly
- AES branch-and-bound solver intact
- State encoding logic intact

## Next Steps
1. Run `master.m` from MATLAB in the v7 root directory
2. Check `output/eq_base.mat` is produced
3. Verify MME_solver reports monotonically decreasing Delta → < 1e-3
4. Verify validate_mme reports bound < 0.05


---
**Context compaction (auto) at 17:37**
Check git log and quality_reports/plans/ for current state.
