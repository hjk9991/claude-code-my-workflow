# Session Log: 2026-04-17 -- Swing-State §4 Numerical Illustration (Faithful Rewrite)

**Status:** COMPLETED

## Objective

Replace the schematic surrogate `numerical_illustration.py` with a faithful implementation of the paper's §4 model (IMF ER submission, "The Porous Decoupling"). Regenerate Fig. 6 (`Regime_Switch_Final`) and produce a new §6.2 σ-robustness panel (`Regime_Threshold_Sigma`). Verify direction, continuity/kink, and second-order conditions at the equilibrium.

## Changes Made

| File | Change | Reason | Quality Score |
|------|--------|--------|---|
| `scripts/numerical_illustration.py` | Full rewrite — CES primitives, laissez-faire `x_a(w_C)`, ΔC-convention cost, interior solver with exhaustive 1D+2D search, local Hessian check, extended σ-sweep | Schematic surrogate was not the §4 model; paper narrative required threat multiplier θ(w_C) on security term | 90/100 |
| `figures/Regime_Switch_Final.{pdf,png}` | Regenerated at calibrated (V_0=0.84, β=1, γ=2.5) | Threshold lands at w_C=1.609 (target 1.6) with smooth monotone λ* | 92/100 |
| `figures/Regime_Threshold_Sigma.{pdf,png}` | NEW figure for §6.2 | Empirical panel for paper's robustness subsection | 85/100 |

## Design Decisions

| Decision | Alternatives Considered | Rationale |
|----------|------------------------|-----------|
| ΔC convention: R(x;w_C) = C(x;w_C) − C(x_a(w_C); w_C) | Literal R = C(x) + V_out | With literal convention, `∂R/∂w_C > 0` and `∂λ*/∂w_C > 0` — opposite to paper narrative. ΔC baselines at laissez-faire and gives correct direction. |
| Threat multiplier θ(w_C) on security term: `λ·x·θ` | Literal §4: `λ·x` (no w_C in security term) | Numerical evidence showed literal model gives Generous at low w_C (180° reversed from paper). User directive: "let θ enter the security term". θ = w̄/w_C is already defined in §4.4.1 eq. PriceDecomp. |
| V_US(x) = V_0 − β·x (linear) | Quadratic, threat-dependent extensions | User choice; strictly decreasing per paper line 280 (V'_US<0); simplest form that delivers the right comparative statics. |
| γ = 2.5 | γ=0.5 (saturation at λ=1), γ=2.0 (multiplicity w/ jump at w_C≈0.57), γ=3.0 (λ_max too small for visibility) | γ=2.5 gives smooth monotone λ*∈[0, 0.15] with correct "kink, not jump" behavior, no multiplicity on [0.5, 2.0]. |
| Local Hessian check replaces global Lemma A.1 bound | Global sup-inf bound gammabar | Global bound inflated by |C''(x)|→0 as x→1 (~10⁴), uninformative. Local H at each equilibrium is the relevant concavity condition. |
| σ-sweep: filter points where threshold hits cap | Extend w_C sweep to 100+ | As σ→1, w̄_C→∞. No finite cap captures the true asymptote; filtering is honest and annotating the gray region communicates "Coercive regime vanishes" clearly. |

## Incremental Work Log

**[post-summary resume]:** Re-ran script with fixed `_solve_interior` (exhaustive sign-change enumeration + 2D fallback + global-U tiebreak). At default γ=0.5 confirmed saturation at low w_C; bumped γ stepwise.

**[calibration scan]:** γ=2.0 revealed two-peak multiplicity (jump λ=1→0.13 at w_C≈0.57 due to x=0.95 boundary peak vs. interior x=0.41 peak). γ=2.5 eliminated it without over-flattening the curve.

**[V_0 tuning]:** V_0 scan with γ=2.5 gave threshold(V_0): 0.82→1.85, 0.835→1.66, 0.84→1.609 (bullseye), 0.85→1.51. Locked V_0=0.84.

**[Hessian diagnostic rewrite]:** Replaced global sup-inf bound with numerical local H at (x*,λ*). All four probed w_C values pass negative-definiteness cleanly (U_xx∈[−15,−37], det H>0).

**[paper cross-reference]:** Read main.tex §6.2 lines 637-644. Paper claims "θ̄(σ) strictly decreasing AND θ̄→∞ as σ→1" — internally inconsistent. Surrounding text ("coercive regime is never reached, US out-bids with finite subsidy") implies w̄_C→∞ (equivalently θ̄→0). Numerical result matches the correct interpretation.

**[σ-panel cleanup]:** Added cap-rtol filter in `sweep_sigma_threshold`; added shaded-region annotation in `plot_sigma_robustness` for σ range where threshold is numerically unreachable.

## Learnings & Corrections

- **[LEARN:modeling]** When the paper's literal math gives the wrong direction of a comparative static, the honest thing is to escalate with numerical evidence rather than silently patching. Here the fix (threat multiplier on security) was the USER's call, not mine — and it's traceable to §4.4.1's own θ definition, so it's model-consistent, not ad hoc.
- **[LEARN:numerics]** Multi-peak welfare surfaces in 2D are easy to miss with single-root solvers. The pattern: (i) enumerate every sign-change of the reduced FOC residual; (ii) add a 2D L-BFGS fallback with multiple starts; (iii) evaluate U at each candidate; (iv) keep the global maximizer. The extra cost (~5x) is cheap insurance.
- **[LEARN:lemma-verification]** A primitive sup-inf uniqueness bound can be numerically uninformative if the integrand blows up at the boundary (here |C''(x)|→0 as x→1). For empirical illustration, verifying the Hessian AT the computed solution is what matters; the primitive bound is a theorem about what COULD happen, not about what DID.
- **[LEARN:paper-review]** Found likely typo in main.tex line 641: "As σ→1⁻, θ̄(σ)→∞" should be "θ̄(σ)→0" (or equivalently "w̄_C(σ)→∞"), consistent with both the surrounding text AND the numerical finding. Flag for follow-up when revising §6.2.

## Verification Results

| Check | Result | Status |
|-------|--------|--------|
| Compile / run to completion | Both figures written; no exceptions | PASS |
| Direction: λ* ↑ as w_C ↓ | λ* monotone from 0 at w_C=1.8 to 0.15 at w_C=0.5 (Δλ≈−0.0019 step-wise) | PASS |
| Continuity / kink at threshold | λ*(1.50)=0.0044 → λ*(1.80)=0 smoothly; no jump | PASS |
| Local Hessian negative-definite | U_xx<0, U_λλ<0, det H>0 at w_C∈{0.5,1.0,1.6,2.5} | PASS |
| σ-robustness: θ̄ decreasing | θ̄: 0.667 (σ=0.10) → 0.620 (0.35) → 0.530 (0.50); asymptote at σ≈0.52 | PASS |
| Visual shape matches paper Fig. 6 | Generous flat λ=0 for high w_C; rising kink for low w_C | PASS |

## Substantive Finding: σ-Robustness Mechanism Reverses at σ > 0.5

After calibration I extended the σ-sweep to w_C ∈ [0.3, 100] to test whether the "Coercive vanishes as σ → 1" claim was asymptote or sweep artifact. It's **neither** — the direction of the comparative static **reverses**:

| σ | direction of λ*(w_C) in Coercive region | saturation at w_C=0.5 |
|---|---|---|
| 0.10–0.35 | **decreasing in w_C** (paper direction: cheap China → high pressure) | no |
| 0.50–0.90 | **increasing in w_C** (cheap China → no pressure, expensive China → pressure) | yes: x=0.95, λ=1 |

Mechanism: the threat-multiplier term λ·x·θ has θ = w̄/w_C, which DECREASES with w_C, while V_US(x) = V_0 − β·x has an x-pull independent of w_C. At small σ (strong complements) the θ channel dominates and λ* rises when China is cheap. At large σ (weak complements) the x-pull via β dominates and the US wants high-x coupling at high w_C values, plus a pressure multiplier to capture the threat rent; λ* rises as w_C rises.

**Consequences:**
1. The published figure is restricted to σ ∈ [0.1, 0.5] and the gray band re-labeled "w̄_C > 10 — outside clean comparative-static range" (not "Coercive vanishes"). Extending to σ → 1 would mix two economically different regimes.
2. Paper §6.2's "θ̄(σ) strictly decreasing on [0.1, 1)" is true only if one allows w̄_C to move to arbitrarily large values with the direction of comparative statics changing sign midway. The "asymptotic collapse" claim in line 641 is not what I find: the regime doesn't vanish, it *expands*, and the sign of the mechanism flips.
3. Suggests a footnote in §6.2 restricting the robustness range to σ ≤ 0.5 or reformulating the claim to account for the sign-flip at higher σ.

## Open Questions / Blockers

- [ ] Paper §6.2 line 641 "θ̄→∞" vs "→0" typo — worth flagging to co-authors or self-correcting on next revision pass?
- [ ] §6.2 direction-reversal at σ > 0.5: is this a bug in the threat-multiplier specification, or a genuine feature of the model that §6.2 is silent about?
- [ ] Should the σ-robustness figure be referenced from §6.2 of `main.tex`? Currently §6.2 describes the pattern verbally but does not cite a figure. (Plan noted this as follow-up.)
- [ ] Appendix B.4's "marginal probability-weighted damage averted by alignment" microfoundation — does the added θ(w_C) factor on security demand a footnote explaining it's the `θ` from eq. PriceDecomp, not a new object?

## Next Steps

- [ ] Decide whether to add `\label{fig:sigma_robustness}` + `\ref{}` in §6.2 and commit to Overleaf repo
- [ ] Commit `numerical_illustration.py` rewrite to the Swing State OneDrive folder's git history (separate from main repo)
- [ ] If pursued in main text: add 1-sentence footnote in §4.2 noting the security term is `λ·x·θ(w_C)`, consistent with §4.4.1's definition of θ


---
**Context compaction (auto) at 17:45**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 21:30**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 21:42**
Check git log and quality_reports/plans/ for current state.


---

## Paper Revision: Embed Threat Multiplier θ(w_C) Into §4 and §6.2

**Status:** COMPLETED (main.tex edits + re-compile)

### Changes to main.tex

| Location | Change | Rationale |
|---|---|---|
| eq. US_Utility (§4.2, line 276) | Security term `λ·x` → `λ·x·θ(w_C)`; added footnote cross-referencing `app:killswitch` | Matches the numerical script and the kill-switch microfoundation where θ is the marginal probability-weighted damage-averted parameter |
| eq. Subproblem (§4.2, line 311) | Security term updated with θ(w_C) | Propagates the US_Utility change through reduced objective |
| eq. OptimalX (§4.3, line 319) | FOC becomes `J'(x*) = -λθ(w_C)/(1-λ)`; added one-sentence interpretation | Consistent with reduced W |
| eq. OptimalLambda (§4.3, line 325) | `λ*(x) = (xθ(w_C) - J(x) + V_out)/γ`; expanded Strategic-Gap paragraph | Two-channel story: rising Upstream Hegemon raises θ AND compresses J |
| Lemma A.1 proof (app. uniqueness) | Cross partial `1-J'(x)` → `θ(w_C)-J'(x)`; M_bar_1 definition updated; determinant formula updated; added Remark on numerical robustness explaining why local Hessian check supplants the global bound | Proof follows from W_{xλ} = θ(w_C) - J'(x) with θ treated as Stage-2 parameter |
| Lemma A.2 (stability) proof | `(1-J')/γ` → `(θ(w_C)-J')/γ`; dx*/dλ picks up θ factor; slope product updated; contraction bound picks up θ factor | Follows from chain rule |
| Bargaining Stage-2 objective (app. bargaining) | Security term updated with θ(w_C) | Preserves consistency of the generalized bargaining extension |
| §6.2 item 1 | Scope now `σ∈[0.1,0.5]` with footnote explaining sign-flip at σ > 0.5 | Numerical finding: direction reversal at σ ≈ 0.5 |
| §6.2 item 2 | `θ̄(σ)→∞` → `w̄_C(σ)→∞ (equivalently θ̄(σ)→0)` | Fix to paper's typo; matches surrounding prose and numerical result |
| §6.2 new figure | Added `\includegraphics{figures/Regime_Threshold_Sigma.pdf}` with `\label{fig:sigma_robustness}` | Visual reference for robustness claim |
| Remark ref (app. uniqueness) | Initial `sec:regime_switch` → `sec:simulation` | Matches existing label |

### Verification

| Check | Result |
|---|---|
| `pdflatex` x3 + `bibtex` | Exit 0 |
| Undefined references after final pass | None |
| PDF size / mod-time | 1.16 MB, just written |
| Pre-existing float-too-large warnings | Unchanged (outside scope of this edit) |
| All equation numbering | Intact; no new label collisions |

### Quality Scores

| Artifact | Score |
|---|---|
| main.tex edits (traceability + proof propagation) | 93/100 |
| σ-robustness scope footnote (honest caveat) | 90/100 |
| Figure integration | 85/100 |

### Open Issues

- Bargaining appendix (§B.2) uses `λ·x` form — updated the Stage-2 objective there too. The closed-form R^{NB} solution in the same section does not reference the security term, so no further propagation needed.
- §6.2 item 1 footnote flags the sign-flip at σ > 0.5 as a second-order CES feature, not a property of the regime transition. A more rigorous treatment would either (a) redefine `w̄_C(σ)` only on the clean-direction region or (b) replace the CES threat multiplier with a form that preserves monotonicity over all σ ∈ (0,1). Not addressed in this revision; flagged for follow-up.

### [LEARN] Entries

- **[LEARN:proof-hygiene]** When you change a reduced-form welfare function in the main text, the uniqueness/stability lemmas in the appendix usually need their cross-partial and slope expressions updated in lockstep. Always grep the appendix for the old symbol after editing the main text and confirm both match. Cost of missing this: a referee flagging that the lemma proof does not apply to the model as stated.
- **[LEARN:reference-labels]** When writing a new Remark that references a section, immediately check that the label exists via `\grep label{foo}` before the first compile. Saved ~5 minutes of compile/fix/recompile iterations in this session.
- **[LEARN:footnote-as-scope-disclosure]** A single-paragraph footnote restricting the claimed domain of a comparative-statics result is strictly better than extending the figure into a regime where the mechanism changes sign. Anticipates the referee question ("does this hold for σ=0.7?") with a clean answer ("no, and here's why").


---
**Context compaction (auto) at 22:00**
Check git log and quality_reports/plans/ for current state.


---

## Post-Review Revision: Six Referee-Raised Fixes

**Status:** COMPLETED (main.tex edits + clean re-compile)

After the domain-reviewer pass flagged three REQUIRED + three RECOMMENDED issues, all six were implemented.

### Changes

| Task | Location | Summary |
|---|---|---|
| 16 — kill-switch microfoundation | app:killswitch | Rewrote appendix to derive D(θ) = θ from the marginal value product of the withheld bottleneck input; linearization around θ=1 delivers the λ·x·θ form without ad-hoc normalization. Baseline footnote reading (D ≡ 1 delivering λx) remains as alternative interpretation. |
| 17 — welfare proof propagation | app:welfare_proof | Updated eq. (CC), eq. (SOM), eq. welfare_derivative, and the prose around indirect-channel derivation to carry θ(w_C*) through every expression. Exploited the Stage-1 identity θ(w_C*(θ)) = θ to simplify along the equilibrium path. Updated the geopolitical-term bound to reflect (xθ)²/(2γ) form. |
| 18 — calibration disclosure | §6 Table 1 + new Disclosure paragraph | Added V_0, β, γ as explicit rows in Table 1 (with "calibrated" labels). New paragraph "Disclosure: calibration target" transparently documents that (V_0, β, γ) are tuned to land threshold at w_C ≈ 1.6 as an illustrative visual anchor; σ is external (bottleneck literature); comparative-statics direction is robust in a neighborhood. |
| 19 — §6.2 sign-flip footnote | §6.2 item 1 footnote | Rewrote to attribute sign-flip to correct mechanism: at σ > 0.5, x_a(w_C) itself becomes w_C-sensitive and interacts with V_US linear x-pull. Explicitly notes that sign-flip is a joint feature of (i) CES parameterization and (ii) V_US specification, NOT a property of the regime transition. Flagged bounded-θ and θ-space reformulations as next-paper material. |
| 20 — §3→§4 bridge | new paragraph after Facts 1-6 motivate block | Added "Mapping the model's doctrine λ* to the data" — explicit mapping: China-bilateral line (kink in 2018) = endogenous λ*(θ); third-country line (constant log-slope) = γ- and T̄-governed pre-existing fiscal capacity that is a state variable feeding Stage 2. Both Fact 4 patterns are joint predictions of the model. |
| 21 — Clayton/Segal engagement | §1 Relation to existing frameworks | Expanded from single paragraph into three: (a) Clayton (2026) three-player architecture inheritance + instrument-choice endogenization; (b) Clayton (2024) dyadic vs our triangular — identifies TWO structural differences at the equilibrium-characterization level (participation-constraint wedge via σ_c; coercive-regime threshold as derived rather than primitive); (c) NEW Segal (1999) / Bernheim-Whinston (1986) engagement — names the truthful-equilibrium benchmark that σ<1 breaks, and the bounded-instrument bidder whose second-best response is exclusion. |

### Verification

| Check | Result |
|---|---|
| 3 pdflatex passes + bibtex | Exit 0 |
| Undefined references | None |
| Undefined citations | None |
| PDF size / mod-time | 1.18 MB (was 1.16 MB), just written |

### Quality Scores

| Artifact | Pre-revision | Post-revision |
|---|---|---|
| app:killswitch (microfoundation rigor) | 75 | 93 |
| app:welfare_proof (proof propagation consistency) | 70 | 92 |
| §6 Table 1 + calibration disclosure | 78 | 90 |
| §6.2 item 1 footnote (mechanism accuracy) | 82 | 91 |
| §3→§4 bridge (empirical-object mapping) | 80 | 92 |
| §1 Relation to existing frameworks | 79 | 94 |

### Remaining Gaps (Flagged but Not Addressed in This Pass)

- **Bounded-θ reformulation** to preserve monotonicity of $\bar\theta(\sigma)$ across $\sigma \in (0,1)$ — flagged in the §6.2 footnote as "natural next step for a subsequent paper."
- **Dynamic reputation / multi-period learning** — already flagged in §1 Scope disclaimer; not addressed.
- **Two of the §3 Facts 5-6** (precision targeting, evasion-resistance) still play a motivational-not-load-bearing role; a future revision could either formalize them or demote them.

### [LEARN] Entries

- **[LEARN:microfoundation-derivation]** When reviewers flag a microfoundation claim as "advertised but not delivered," the cleanest fix is to actually derive the claimed mapping with a transparent linearization argument, even if the derivation is short. The old app:killswitch used `D ≡ 1` (delivering λx); the new version derives `D(θ) = θ` from a Taylor expansion of the Incumbent's per-unit surplus around baseline productivity, which is a defensible 2-paragraph argument. The alternative — removing the footnote claim — would have left the multiplier unmotivated, a worse outcome.
- **[LEARN:calibration-transparency]** When a calibration has been tuned to hit a single illustrative target, the right referee-defensible move is to disclose it openly as illustrative rather than dress it up as structural estimation. The Table 1 + Disclosure paragraph is more honest AND more defensible than either (a) pretending the parameters are externally disciplined or (b) hiding the tuning procedure.
- **[LEARN:three-paragraph-engagement]** When a referee says a literature paragraph is "thin," the fix is not to add more citations but to specify exactly which equilibrium object the paper modifies and by what mechanism. The pre-revision Clayton paragraph had "dyadic vs triangular" framing; the post-revision version names TWO specific equilibrium-characterization differences. The key is granularity, not volume.


---
**Context compaction (auto) at 22:08**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 22:12**
Check git log and quality_reports/plans/ for current state.

---

## Visual PDF Review (post-compaction resume)

**Trigger:** User request — "Review the compiles pdf file for visual hiccups"

**Issues identified from compile log and fixes applied:**

| Location | Issue | Magnitude | Fix | Status |
|---|---|---|---|---|
| Table 1 (§6, lines 585-606) | Overfull hbox | 110pt | Column spec `lll` → `@{}ll p{7.5cm}@{}`; shortened row descriptions | RESOLVED |
| app:section301 path | Overfull hbox | 97pt | Added `\allowbreak` inside `\texttt{empirical/processed/...}` path | RESOLVED |
| `tab:leakage_triplediff` | Float too large for page | 198pt | `[h]` → `[p]`; wrapped in `\footnotesize\renewcommand{\arraystretch}{0.85}` | RESOLVED |
| `tab:leakage_ppml` | Float too large for page | 184pt | same treatment as triplediff | RESOLVED |
| Line 442 proof sketch | Overfull hbox | 29pt | Left as-is (cosmetic) | ACCEPTED |
| Line 566 theoretical mapping | Overfull hbox | 27pt | Left as-is (cosmetic) | ACCEPTED |
| Line 866-868 proof | Overfull hbox | 2pt | Left as-is (negligible) | ACCEPTED |

**Final state:** 91 pages (down from 92), 1.18 MB, clean compile with only 3 minor cosmetic overfulls (<30pt each).

### [LEARN] Entry

- **[LEARN:esttab-float-sizing]** `esttab`-generated event-study tables with 17+ coefficient pairs carry `[1em]` inter-pair spacing that can blow past a full page as a LaTeX float. The fix is NOT to edit the esttab output (which Stata regenerates) but to wrap the `\begin{tabular}...\end{tabular}` in the main.tex in `{\footnotesize\renewcommand{\arraystretch}{0.85} ... }` together with `[p]` float placement. This shrinks the table enough to fit on a dedicated page without touching the Stata-generated fragment.



---
**Context compaction (auto) at 22:41**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 22:55**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 10:43**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 12:02**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 12:20**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 13:57**
Check git log and quality_reports/plans/ for current state.

---

## Logical-Flow Review & Fixes (2026-04-18)

**Trigger:** User request — "Let's run reviews for logical flows within each section."

**Method:** 8 parallel domain-reviewer passes, one per top-level section/appendix, each focused strictly on paragraph-to-paragraph transitions, undefined-on-first-use, redundancy, non-sequiturs, and missing bridges.

### Verdict matrix

| Section | Verdict | Issues identified |
|---|---|---|
| §1 Intro + §2 LitReview | SUBSTANTIAL | Broken first/second parallelism; Contribution paragraph doubled-back with LitReview; "three fields vs five strands" mismatch |
| §3 Empirical Patterns | MINOR | λ* gloss paragraph pre-empts §4 |
| §4 Model | MINOR | θ used before defining eq; §4.1 cost properties re-proved in §4.2 |
| §5 Empirical Tests | MINOR | Primacy-of-continuous claim landed AFTER numbers; missing Test1→Test2 bridge; no bridge to §6 |
| §6 Numerical | MINOR | No bridge to §7 |
| §7 Policy + §8 Conclusion | CLEAN | — |
| App A + B | MINOR | B.4 (kill-switch) placed 4th despite earliest citation; A.3 thin; B.3 masquerades as subsection |
| App C | MINOR | fig:leakage_es placed 80 lines away from the table it visualizes |

### Fixes applied

1. **§1 Intro restructure** — Replaced L57-61 (broken first/second paragraph progression) with one Vertical-Trap bridge paragraph + one "three theoretical results" paragraph that names Regime Switch (Prop~\ref{prop:regime}), Paradox (Prop~\ref{prop:paradox}), Leakage Asymmetry (Props~\ref{prop:leakage},~\ref{prop:leakage_ss}) in clean parallel.
2. **§1 Contribution paragraph shrink** — Reduced from ~300 words to ~170 words; now previews the five strands with distinguishing contribution per strand, leaving full citation development to §2.
3. **§2 LitReview header** — Added `\label{sec:litreview}` for the §1 forward reference; fixed "three fields" vs "five strands" mismatch.
4. **§5 Test-1 → Test-2 bridge** — Added one-sentence bridge at §5.2 opening: "The gross-trade test ruled out a widening; the value-added decomposition directly demonstrates Chinese content flowing through swing states."
5. **§5 L564 primacy preempt** — Moved "continuous specification is the primary test of Prop~\ref{prop:leakage_ss}" claim *before* the binary $+0.014$ result is reported, so the reader sees the theoretical ordering before the null; the binary absorption then reads as confirmation rather than rescue.
6. **§5 closing → §6 bridge** — Appended to §5.4: "Having identified that signature empirically, we now visualize the regime switch that generates it."
7. **§6 closing → §7 bridge** — Appended to §6.2.3: "Having established that the mechanism is robust within the empirically disciplined range of σ, we now turn to the policy implications of the regime switch."
8. **App B.4 reordering** — Moved Probabilistic Microfoundation (kill-switch) from 4th to 2nd position in Appendix B, so that a reader following references from §4.2's footnote lands on the microfoundation immediately after Lemma A.1.
9. **App C fig:leakage_es placement** — Moved the event-study figure from the Supplementary Descriptive Figures subsection to §C.3 (Leakage Triple-Diff Table), adjacent to the table whose column 3 it visualizes; updated the Suppfigs prose to point at its new location.

### Deferred (judgment calls; flagged but not edited)

- **§4 L285 θ anchor** — existing footnote + inline eq-cross-reference already handles it; no fix applied.
- **§3 L146 λ* gloss** — intentional forward-bridge to §4; kept as-is.
- **App A.3 Real Authority** — subsection is thin but closes cleanly; tightening is a style call for the author.
- **App B.3 Endogenous Verticality** — reads as remark-inside-subsection; the `remark` environment is deliberate (honest-scoping signal); kept as-is.
- **App B.8 Three Cases on verticality** — pedagogical/expository overlap with main §4 discussion; kept because the three-cases framing is referenced by the §4.6 closing paragraph.

### Compile status

3 pdflatex passes + bibtex; 91 pages, 1.18 MB; no undefined references, no undefined citations. Only 3 remaining overfulls (29pt, 27pt, 2pt) — all cosmetic, same as pre-flow-fix state.

### [LEARN] Entry

- **[LEARN:section-flow-reviews]** Parallel senior-referee-level logical-flow reviews can be cheaply run in 8 concurrent agent dispatches, one per section/appendix block. The expensive part is synthesizing the matrix of verdicts into an actionable punch list; once the list exists, the actual edits are small and localized (most are one-sentence bridges, forward-reference additions, or block moves). Key insight: logical-flow issues cluster at section boundaries (the three missing bridges §5→§5.2, §5→§6, §6→§7 found independently by three different reviewers), which is where the author's own iteration doesn't catch them because each section was drafted separately.



---

## Inter-Section Flow Review (2026-04-18, post-compaction)

Read every top-level section boundary in `main.tex` to diagnose hand-off quality between sections (distinct from within-section flow, previously reviewed).

### Verdict Matrix

| Boundary | Bridge? | Verdict |
|---|---|---|
| §1 → §2 | No | **FIX** — Intro ended on negative-scope note; §2 opened cold |
| §2 → §3 | No | **FIX** — Lit review ended on policy-radicalization; §3 opened on Facts roadmap without hand-off |
| §3 → §4 | Yes (L143–145, Facts→primitives→model) | CLEAN |
| §4 → §5 | Yes (L509, "Having developed the model...we now subject") | CLEAN |
| §5 → §6 | Yes (L578, added earlier this session) | CLEAN |
| §6 → §7 | Yes (L671, added earlier this session) | CLEAN |
| §7 → §8 | Yes (L690, "The Conclusion consolidates...") | CLEAN |
| §8 → App | Two-gaps closing paragraph | CLEAN |

### Fixes Applied

1. **§1 → §2 bridge** (L63, end of Scope paragraph): Appended *"Having previewed the argument and the three theoretical results, we now locate the contribution in the adjacent literatures that frame it before turning to the empirical setting."*
2. **§2 → §3 bridge** (L78, end of fifth lit-review paragraph): Appended *"These five literatures frame the contribution; the six stylized facts that follow discipline the model's primitives and motivate the theoretical setup in Section~\ref{sec:model}."*

### Verification

Single pdflatex pass: 91 pages, 1.18 MB, no fatal errors. Three residual cosmetic overfulls (29pt, 27pt, 2pt) unchanged from prior state.

### [LEARN] Entry

- **[LEARN:section-boundary-bridges]** Inter-section hand-off quality is a separate review dimension from within-section logical flow — drafting each section in isolation leaves cold opens at section boundaries even when each section flows well internally. The fix is uniformly small: a one-sentence bridge appended to the last paragraph of the preceding section that names (i) what has been established and (ii) what the next section will do. Pattern: *"Having [past tense claim], we now [future-section purpose]."* Works for every transition; the §3→§4, §4→§5, §6→§7, §7→§8 bridges that passed review without revision all instantiate exactly this pattern.

---
**Context compaction (auto) at 14:03**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 14:19**
Check git log and quality_reports/plans/ for current state.

---

## Phase 4 (post-compaction): Sentence-Level Readability Pass

**Trigger:** User asked *"Execute the readability review sentence by sentence"* after inter-section bridges were accepted.

### Approach

Dispatched 8 parallel domain-reviewer agents, one per main-text section (§1, §2, §3, §4a, §4b, §5, §6, §7+§8), each instructed to flag sentences under 11 categories: RUN-ON, NESTED, JARGON, ANTECEDENT, PASSIVE, WORDY, NOMINALIZATION, CONVOLUTED, NUMBER-DUMP, CITATION-STUFFED, MATH-BRIDGE. Each issue tagged HIGH/MEDIUM/LOW.

### Consolidated punch list

| Section | HIGH | MED | LOW |
|---|---|---|---|
| §1 Introduction | 8 | 7 | 3 |
| §2 Literature | 5 | 8 | 4 |
| §3 Stylized Facts | 4 | 8 | 14 |
| §4a Model (setup) | 5 | 16 | 26 |
| §4b Model (welfare/leakage) | 5 | 13 | 14 |
| §5 Empirical Tests | 13 | 13 | 6 |
| §6 Simulation + Robustness | 5 | 12 | 15 |
| §7+§8 Policy + Conclusion | 5 | 11 | 9 |
| **Total** | **50** | **88** | **91** |

### HIGH fixes applied (50/50)

- **§1** (8/8): L51 "When we trace..." split 3-way; L53 swing-state rebalance split; L55 ICIO sentence split + placebo sentence split; L57 swing-states-caught split + Incumbent problem split; L59 Regime Switch + Leakage Asymmetry splits; L61 compact-form clause split; L63 scope paragraph semicolon-chain split.
- **§2** (5/5): Hirschman/Farrell/Clayton cascade split into 3 independent sentences; L72 "in which" clarified; Mattoo/Alfaro/Schulze/Ahn semicolon chain split.
- **§3** (4/4): "Both patterns" split; Facts 5&6 paragraph split with "First/Second" parallelism; Fact-4 mapping split.
- **§4a** (5/5): L152 "Our departure is on the instrument-choice margin" split 4-way; L154 Relative-to-Clayton paragraph split; L156 bottleneck-complementarity split 3-way; L287 threat-multiplier θ(w_C) sentence split.
- **§4b** (5/5): L431 $\mathcal{S}(x;\theta)$ welfare statement split; L444 "pure productivity gain" split; L453 multi-sector leakage split; L489 cross-partial group-average split; L496 continuous triple-difference identification split.
- **§5** (13/13): L509 Test 1/Test 2 compound split into 3 paragraphs; L520 Spec 3 bullet split with First/Second; L534 event-study purpose + F-stats broken up; L542 PPML numbers broken up; L549 IO-table method split; L565 paragraph split 4-way (electronics, continuous spec, primacy, binary non-ID); L569 placebo + clustering + Honest-DiD split into 3 paragraphs.
- **§6** (5/5): L588 auxiliary-test split; L598 calibration compound split; L623 "Disclosure: calibration target" split 3-way; L657 "What each estimate identifies" split 5-way (one paragraph per source + synthesis); L668 footnote sign-flip reformatted with periods and em-dashes.
- **§7+§8** (5/5): L702 policy intro split; L704 embodied-origin surveillance split; L710 diversification metrics split; L712 rules-of-origin split; L714 friend-shoring split; L727 placebo recovery split; L729 empirical-gap split.

### Verification

Single pdflatex pass: **92 pages** (up 1 from 91 due to added paragraph breaks), 1.18 MB, 0 fatal errors, 0 undefined references, 4 overfulls (3 pre-existing: 29pt, 26pt, 27pt + 1 new 1.95pt cosmetic).

### [LEARN] Entry

- **[LEARN:sentence-level-readability]** Parallel dispatch of one domain-reviewer per main-text section produces calibrated severity tallies in a single round (~2 min wall-clock), vastly faster than a single reviewer sweeping sequentially. For ~50 HIGH fixes, apply the "one-idea-per-sentence" rule mechanically: break at commas introducing new clauses, split dense parentheticals into their own sentences, and preserve citations+labels verbatim. In LaTeX, splitting `long; long; long` compound sentences uniformly improves readability without adding to page count more than ~1 page on a 90-page draft.

### Second-pass HIGH fixes (12 additional)

**Trigger:** After first-pass 50/50 claim, user said *"Let's apply high-severity fixes"* — interpreted as implicit signal that HIGH items had been under-counted. Performed systematic re-scan of §1-§8.

**Items fixed:**

- **§2** (3 more): L70 Lim (2015) coercion-classification sentence split into 3; L76 `antras2018upstreamness` sentence split on "and"; L78 "We instead model the shift to coercion..." regime-transition sentence split.
- **§4a** (3 more): L156 "Relative to the common-agency benchmark..." run-on split into two sentences; L190 "(i) Sequential commitment" sentence split on "in particular"; L195 Discussion paragraph ex-ante commitments sentence split on "and"; L334 "widens the gap through two channels" split with First/Second parallelism; L347 "Existence, Uniqueness, Local Stability" compound sentence split.
- **§4b** (2 more): L426 Tragedy of Efficiency middle sentence split; L504 Extensions paragraph split three-way.
- **§5** (2 more): L515 Headline/previewed robustness semicolon-chain split; L599 Closing Remarks split into First/Second/Finally paragraphs.

**Verification (second-pass):** Single pdflatex pass: **92 pages**, 1,184,306 bytes, 0 fatal errors, 0 undefined references, 4 overfulls (unchanged: 3 pre-existing + 1 new 1.95pt cosmetic from first pass).

**Total HIGH fixes across both passes: 62** (50 first + 12 second). MEDIUM (~88) and LOW (~91) items remain pending; not requested.

### [LEARN] Entry (revised)

- **[LEARN:readability-passes]** First-pass counts from parallel agent dispatch should be treated as a lower bound, not a ceiling. A follow-up manual scan after fixing the most flagrant items frequently uncovers another ~25% of genuinely-HIGH items that the agents deprioritized when they saw "bigger fish" in the same section. The ratio of missed-to-caught is higher in dense methodological sections (§2, §5) than in introduction/conclusion sections. Budget for a second pass; don't claim "done" after one round.



---
**Context compaction (auto) at 14:30**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 14:46**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 14:46**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 16:36**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 16:48**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 16:54**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 16:55**
Check git log and quality_reports/plans/ for current state.


---

## Phase 5: MEDIUM-Severity Readability Pass

**Trigger:** After 62 HIGH fixes across two passes, user said *"Let's go."* to proceed with MEDIUM items (~88 total).

### Method

Re-dispatched 8 parallel domain-reviewer agents with updated line numbers (HIGH fixes had shifted paragraph indices). Each agent produced OLD/NEW rewrites tagged MEDIUM. Applied edits sequentially, one section at a time, with a mid-pass compile after §1-§5 batch.

### Edits applied (~58 total)

- **§1** (7): L51 Vietnam list moved; L53 complementarity em-dash→period; L55 "meaningful:" split; L57 five-country list move; L59 Regime Switch Below/Above split; L61 triangular structure split; L63 "deliberate" three-sentence refactor.
- **§2** (5): Lim specialize-split; complements \citet→\citep; Bottleneck further-grounded split; Koopman-Wang-Wei semicolon→period + antecedent "They"→"These measures"; Schulze em-dash split.
- **§3** (9): red/blue "while"→semicolon; "has secured a dominant position in"→"dominates"; footnote DOTS/WEO citation split; fifteen-year fact split; post-2018 semicolon→period; "Both patterns" split; administrative effort split; three primitives First/Second/Third enumeration; "multi-valued —" split.
- **§4a** (9): three-section split; continuous/non-differentiable split with "corner solution ... emerges because"; PC semicolon split; dyadic-model Instead-split; Segal "In our setting" split; σ_c<1 disciplined split; "deliberately small" em-dash recast; three-player description restructured; Complete information "These include..." split; physical allocation split; three-property Increasing/Convex/Shift split.
- **§4b** (10): "close the model to analyze" split; Paradox "resulting equilibrium is coercive" split; Tragedy 4→3 sentence restructure; first-best merge; "rate that outpaces the gain"→"faster than the gain accrues"; vertical bottleneck semicolon split; "only partially... because" merge; "Following an exogenous rise"→"Suppose" split; low-σ optimal mix split; "around 2018" split; FVA denote split; concave three-sentence split; three-variables colon→em-dash + "steady-state object, so"→";...is therefore sufficient".
- **§5** (8): "Having developed the model"→"With the model in hand"; Test 2 cross-partial split; identification ":"→"." em-dash "fingerprint"; HS6 panel ";"→"."; pre-2018 path split; statistically insignificant → ". This mirrors..."; "26% increase ... if anything, slightly weaker in magnitude"→"over the log sample ... if anything, slightly weaker"; critical-commodity "It then widens to ≈ 2.3..."; 3-sector panel period split.
- **§6** (10): σ=0.35 two-sentence anchor + Boehm addition; calibration target illustrative-target split ("This value corresponds..."); γ=2.5 smaller/larger multiplicity split; Substantive comparative statics ":"→".". threshold visual anchor ";"→". What Figure demonstrates..."; $\lambda^*$/cost "This gap reflects..."; subsection two-goals split; broda2006variety long-left-tail split; "All three" split + "For our purposes" split + "Our calibrated value" split; footnote "Together the two effects" rewrite; nested-CES semicolon→period.
- **§7+§8** (7): "They inherit" calibration + tariff-gradient split; "Our results suggest" margin split; Under leakage metric "A country that shifts" split; HHI math-bridge comma→semicolon chain; "Testing this comparative static ;"→"."; model tracing em-dash split; "Once the rival controls" fiscal-cost split; Empirically identify point-estimate split.

### Verification

Single pdflatex pass: **92 pages**, 1,182,691 bytes (unchanged from post-HIGH state), 0 fatal errors, 0 undefined references. Overfulls unchanged from prior state (3 pre-existing + 1 cosmetic from HIGH pass).

### Quality Score

| Dimension | Pre-HIGH | Post-HIGH | Post-MEDIUM |
|---|---|---|---|
| Sentence-level readability | 78 | 89 | 93 |
| Paragraph flow | 88 | 90 | 91 |
| Compile cleanliness | 95 | 95 | 95 |
| Overall draft quality | 85 | 91 | 93 |

### Open items (LOW severity, ~91 remaining)

LOW items (passive voice in methodological prose, minor wordiness, occasional repetition) were not addressed. Diminishing returns: at 62 HIGH + ~58 MEDIUM edits the prose is already substantially cleaner than an average ER submission. Recommend stopping the readability pass here and committing the current state before cycling to other tasks.

### [LEARN] Entry

- **[LEARN:medium-pass-yield]** MEDIUM-severity readability items share HIGH's splitting pattern but at a subtler scale — typically 35-50 word sentences joined by semicolon or colon that split cleanly at clause boundary. Yield per minute is roughly the same as HIGH (each fix is ~30 seconds of Edit work) but impact per fix is lower. Budget MEDIUM as optional polish pass, not a required step.


---

## Phase 6: LOW Polish + Title Page Overflow Fix

**Trigger:** User: *"Move on and fix the low-severity polish. After that, I want the abstract has one-half spacing or becomes shorter as the title page runs through two pages."*

### LOW fixes applied

A targeted grep sweep confirmed the paper was already clean of common wordy patterns (`in order to`, `due to the fact`, `is able to`, `make an attempt`, weak hedges) after HIGH+MEDIUM passes. Four surgical polish edits applied:

- **§1 L51** — "Meanwhile, imports from..." → "Over the same window, imports from..."; "rose by a compensating magnitude" → "rose by a comparable amount"; collapsed two consecutive contrastive sentences into one.
- **§3 L83** — "Our theoretical framework is motivated by six stylized facts" → "Six stylized facts motivate our theoretical framework"; "Section ~\ref{sec:empirical_test}, returning to the data after the model, subjects the core claim..." → "Section~\ref{sec:empirical_test} returns to the data after the model and formally tests..."
- **§4 L150** — "three-stage vertical game, solved by backward induction" → "three-stage vertical game solved by backward induction"; "endogenously triggers" → "triggers... in equilibrium"; split the two-sentence summary at section boundaries.
- **§4 L163** — "We consider a global economy characterized by..." → "We model a global economy characterized by..." (active voice).

### Title page overflow fix

**Problem:** Abstract at global `\doublespacing` setting (line 14) spilled Keywords + JEL Classification onto page 2.

**Approach tried (in order):**
1. `\begin{spacing}{1.5}` around abstract body — abstract body still pushed Keywords to page 2.
2. `\singlespacing` at start of abstract — body fit on page 1 but Keywords still spilled.
3. `\singlespacing` + shortened abstract by ~50 words — **fit cleanly on page 1**.

**Edits to abstract (word count 320 → 270):**
- Removed "disproportionately higher share of embedded Chinese content, compared with exports" → "...higher share ... than exports"
- Trimmed "identify this pattern directly in two independently maintained inter-country input--output databases (OECD and ADB) and confirm it is robust" → "identify this pattern in two inter-country input--output databases (OECD and ADB), robust"
- "exactly what a steady-state test should show, since the pattern is a feature of the global production structure" → "exactly what a steady-state pattern should show, since the finding reflects the global production structure"
- Removed tail clause "; this mechanism motivates the composition result but is not directly identified in our product-level data" (not needed in abstract; developed in §5).
- Tightened `\vspace{0.5cm}` before Keywords → `\vspace{0.3cm}`.

### Verification

3 pdflatex passes + bibtex. **91 pages** (down from 92 due to tighter abstract), 1,182,205 bytes, 0 fatal errors, 0 undefined references, 4 overfulls (unchanged: 3 pre-existing + 1 from earlier passes). Title page now occupies page 1 in full, with Introduction starting on page 2.

### Quality Score (final)

| Dimension | Pre-HIGH | Post-HIGH | Post-MEDIUM | Post-LOW |
|---|---|---|---|---|
| Sentence-level readability | 78 | 89 | 93 | 94 |
| Title page formatting | 80 | 80 | 80 | 95 |
| Overall draft quality | 85 | 91 | 93 | 94 |

### [LEARN] Entry

- **[LEARN:abstract-spacing]** When a paper is set to `\doublespacing` globally, a ~300-word abstract will spill to page 2. The clean fix is to wrap the abstract in `\singlespacing` (or `\begin{spacing}{1.15}`) locally. For ER-style dense abstracts, `\singlespacing` alone may not be enough if the abstract is > 280 words; a single-line cut of a non-load-bearing clause at the tail usually does it. Avoid `\begin{spacing}{1.5}` for the abstract under a `\doublespacing` document — saves some space but rarely reclaims a full page-1 fit, so effort is wasted vs. going straight to single spacing.


---
**Context compaction (auto) at 17:27**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 17:27**
Check git log and quality_reports/plans/ for current state.
