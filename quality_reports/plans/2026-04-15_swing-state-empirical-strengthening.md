# Plan: Swing State Empirical Section Strengthening

**Status:** APPROVED 2026-04-15 — execution underway
**User confirmations:**
- (a) Swing State list approved as baseline; robustness runs with narrow set {KOR, TWN, VNM, MEX, NLD} and wide set (+ {SGP, MYS, THA, IDN, IND, JPN, DEU, HUN, CZE, POL, PHL, BRA}).
- (b) User holds a premium UN Comtrade API token → use Comtrade as primary (HS6, importer-reported, 2000–2024); BACI used only as a reconciliation cross-check.
- (c) §5 will be recalibrated using estimated $\hat\sigma_k$ for the bottleneck sector.
**Date:** 2026-04-15
**Parent plan:** `2026-04-15_swing-state-revision.md` (Phases 1–3 completed)
**Target section:** `main.tex` §3 Empirical Motivation — currently 6 descriptive facts; after this plan, will contain three formal empirical specifications linked one-to-one to model primitives.

---

## Part I — What the Model Actually Asks of Data

The theoretical framework (now at 45pp, 6 propositions) has four parameters the data must either **discipline** (calibrate) or **test** (falsify):

| Symbol | Object | Model role | Empirical target |
|---|---|---|---|
| $\sigma$ | Elasticity of substitution between Red and Blue Chain inputs | Bottleneck condition $\sigma<1$ | **Discipline:** estimate sector-specific $\hat\sigma_k$ |
| $x_{kt}$ | Swing-State alignment (share of non-China inputs) | Agent's choice variable | **Measure:** constructible from trade + I-O data |
| $\lambda_t$ | Hegemon's coercive doctrine | Endogenous intensity of coercion | **Measure:** Entity-List intensity, sectorally resolved |
| $w_{Ct}/w_{At}$ | Efficiency gap | Driver of regime switch via $\theta$ | **Measure:** unit values from customs data |

Three testable predictions:
1. **Bottleneck reality:** $\hat\sigma_k < 1$ for sectors the paper identifies as chokepoints (semiconductors, rare earths, critical minerals), and $\hat\sigma_k \ge 1$ for non-chokepoint manufactures (control sectors).
2. **Paradox of Competitiveness (Prop 2):** At the sector×year panel, $\partial \lambda_{kt}/\partial \theta_{kt} > 0$, controlling for confounders.
3. **Bargaining Amplification (Prop 5):** Coercion intensity is stronger against Swing States with higher structural centrality (high $\beta$).

We do NOT propose a structural break test on aggregate $\lambda$ (explicitly flagged as infeasible in §4.10). We focus on within-sector variation.

---

## Part II — Three Formal Empirical Specifications

### Spec 1 — Sectoral Estimates of the Bottleneck Elasticity $\sigma_k$

**Model-to-data mapping.** The Swing State's unit cost function (§4.1) implies that at an interior interior solution, the relative input demand satisfies
$$\ln\frac{I_{A,kst}}{I_{C,kst}} = \text{const} + \sigma_k \ln\frac{w_{C,kst}}{w_{A,kst}} + \varepsilon_{kst}$$
for country $s$, sector $k$, year $t$.

**Estimator.** Two-stage least squares with instruments for relative prices that shift $w_C$ but not demand:
- **IV 1:** Shift-share predicted exposure à la Autor-Dorn-Hanson, using pre-period sectoral shares interacted with national Chinese export surge.
- **IV 2:** Boehm-Flaaen-Pandalai-Nayar-style supply-disruption instruments (earthquakes, COVID port closures, Ukraine war) interacted with China input share.

**Identification.** Standard Feenstra (1994) / Broda-Weinstein (2006) double-differencing if panel allows; ADH-style IV as fallback.

**Output.** A table of $\hat\sigma_k$ for ~30 HS2 sectors. Predicted pattern: semiconductors (HS 8541–8542), rare earths (HS 2805, 2846), batteries (HS 8507) should have $\hat\sigma_k < 1$; textiles/apparel should have $\hat\sigma_k > 1$ (control).

### Spec 2 — Paradox of Competitiveness (sector × year panel)

**Regression.**
$$\lambda_{kt} = \alpha_k + \delta_t + \beta_1 \cdot \theta_{kt} + \beta_2 \cdot \theta_{kt}\times\mathbf{1}\{\hat\sigma_k<1\} + \Gamma' X_{kt} + u_{kt}$$

- $\lambda_{kt}$: sectoral coercion intensity = count of Entity-List additions in sector $k$, year $t$ (or HHI-weighted tech tagging).
- $\theta_{kt}$: Chinese productivity proxy = China's real revealed comparative advantage (RCA) in sector $k$, or China's share of world exports in $k$.
- Fixed effects: sector $k$, year $t$; cluster at sector.
- **Prediction:** $\beta_2 > 0$ (coercion rises with Chinese efficiency *only* in bottleneck sectors).

### Spec 3 — Leakage Channel (Fact 3 formalized)

**Regression.** Triple-difference on swing-state import share of Chinese inputs:
$$\text{ChinaShare}_{skt} = \alpha_s + \alpha_k + \delta_t + \beta_1 \cdot\text{Tariff}_{kt}^{US\to CN} + \beta_2 \cdot\text{Tariff}_{kt}^{US\to CN}\times \mathbf{1}\{s\in\text{Swing}\} + u_{skt}$$

- $s$: importer country (US, Swing States, RoW); $k$: HS6 product; $t$: year.
- **Prediction:** $\beta_1 < 0$ (US direct decoupling), $\beta_2 > 0$ (Swing States absorb the China export displacement → "leakage").

---

## Part III — Data Sources (Web-Verified)

### Trade data
- **UN Comtrade (PREMIUM API, user has token)** — primary source. HS6 bilateral annual, 2000–2024. No rate limit under premium. Use `comtradeapicall` Python lib; store token in `~/.comtrade_token`.
- **BACI** (CEPII, free download) — reconciliation cross-check. HS6 harmonized bilateral trade, 1995–2022.
  Secondary: `http://www.cepii.fr/CEPII/en/bdd_modele/bdd_modele_item.asp?id=37`

### Input-output structure
- **OECD ICIO 2025 edition** (released Jan 2026, free). 80 economies × 45 sectors, 1995–2022. CSV downloads. Download page:
  `https://www.oecd.org/en/data/datasets/inter-country-input-output-tables.html`
  Use for constructing $I_C/I_A$ backward linkages.

### Coercion data
- **US BIS Entity List** — daily CSV at `https://www.bis.doc.gov/.../consolidated-entity-list` (current snapshot only). For historical additions, use the OpenSanctions archival feed or the project's existing scraped Entity-List history (already used for Figs 4–6).
- **Global Trade Alert** — `https://globaltradealert.org/data-center` — 61,000+ interventions including export controls, sanctions, industrial policy. Free academic download. Complements Entity List with subsidies (for constructing $R$).

### Price/unit-value data
- Unit values at HS6 from **BACI** (value/quantity) — already gives $w_{C,kst}, w_{A,kst}$.
- For robustness: **IMF's Export and Import Price Indexes** and **World Bank Pink Sheet** for commodities.

### Supplemental
- **USITC DataWeb** for detailed US imports at HS10 (requires free registration).
- **WITS** (World Bank) as fallback bulk-download interface.

---

## Part IV — Execution Plan (sequential, with checkpoints)

### Phase A — Data Acquisition (1 day)
1. Create data directory: `~/Library/CloudStorage/OneDrive-개인/Research/Swing State Strategy/empirical/`
2. Download BACI (1995–2022, HS6 bilateral, ~4 GB uncompressed).
3. Download OECD ICIO 2025 (80 economies × 45 sectors).
4. Pull BIS Entity List historical via OpenSanctions API + existing project scrape.
5. Pull Global Trade Alert exports.
6. **Checkpoint:** summary statistics of each dataset → `empirical/output/00_data_summary.md`.

### Phase B — Swing-State Definition & Core Panels (0.5 day)
7. Operationalize "Swing State" = {Korea, Taiwan, Vietnam, Mexico, Netherlands, Singapore, Malaysia, Thailand, India, Japan, Germany}. Robustness set: {ex-ante neutrality score from UN voting alignment}.
8. Build master sector × country × year panel from BACI + ICIO.
9. Build master sector × year coercion panel from Entity List (matched to HS codes via keyword NLP — we already have the pipeline that generated Figs 4–6).

### Phase C — Spec 1: Estimate $\hat\sigma_k$ (1 day)
10. Implement IV regression in Stata (`01_sigma_estimation.do`).
11. Produce sectoral table + sector-ranked plot.
12. **Sanity checks:** (i) tight confidence intervals for high-$N$ sectors; (ii) first-stage F > 10; (iii) sign of $\hat\sigma$ matches prior for control sectors.

### Phase D — Spec 2: Paradox of Competitiveness (0.5 day)
13. Implement panel regression (`02_paradox_test.do`).
14. Plot heterogeneous-treatment coefficient ordered by $\hat\sigma_k$.
15. **Falsification check:** run spec on pre-2015 sample only — $\beta_2$ should be weaker (regime switch not yet active).

### Phase E — Spec 3: Leakage Triple-Difference (0.5 day)
16. Implement triple-diff (`03_leakage.do`).
17. Event-study version around 2018 Section 301 tariffs.
18. **Placebo:** random assignment of "tariff shock" dates 2005–2015.

### Phase F — Write-up (0.5 day)
19. Rewrite §3 to integrate three formal specifications alongside (not replacing) the six stylized facts.
20. Add new subsection §3.3 "Tests of Model Predictions" housing Specs 1–3.
21. Add appendix B with full data construction, IV first stages, robustness tables.
22. Update §5 Numerical Illustration to use the estimated $\hat\sigma$ for the bottleneck sector rather than the calibrated $\sigma=0.35$.
23. **Compile + validate-bib.**

**Total estimated effort:** 4 person-days.

---

## Part V — Files to Create / Modify

Under `~/Library/CloudStorage/OneDrive-개인/Research/Swing State Strategy/empirical/`:

```
empirical/
├── raw/
│   ├── baci/              # HS6 bilateral, 1995-2022
│   ├── icio/              # OECD Inter-Country I-O
│   ├── entity_list/       # BIS Entity List history
│   └── gta/               # Global Trade Alert interventions
├── processed/
│   ├── panel_sector_year.dta
│   ├── panel_country_sector_year.dta
│   └── coercion_sector_year.dta
├── scripts/
│   ├── 00_build_panels.do
│   ├── 01_sigma_estimation.do
│   ├── 02_paradox_test.do
│   ├── 03_leakage.do
│   └── 99_make_tables_figures.do
├── output/
│   ├── tables/            # tex fragments
│   ├── figures/           # pdf
│   └── 00_data_summary.md
└── README.md
```

Under the manuscript folder:
- `main.tex` — rewrite §3, add §3.3 and Appendix B.
- `ref.bib` — add Autor-Dorn-Hanson 2013, Feenstra 1994 (if missing), ADH-type shift-share methodological refs.

---

## Part VI — What We Will NOT Do (Out of Scope)

- We will not estimate a full structural model (reserved for R&R).
- We will not build a causal identification strategy for the *choice* of $\lambda$ — the best we can do is test the conditional correlation predicted by Prop 2 and Prop 5.
- We will not attempt network-topology estimation of $\beta$ (bargaining power) — the Prop 5 test uses a proxy (sectoral centrality in ICIO).

---

## Part VII — Risk Register

| Risk | Mitigation |
|---|---|
| BACI 2023/2024 not yet released | Use 2022 as end-year; Comtrade as extension |
| ICIO 2025 sector bridge to HS6 imperfect | Use CEPII sector concordance; document mapping |
| Entity List sectoral tagging noisy | Use the existing keyword NLP pipeline + manual validation on top 100 entities |
| IV first-stage weak | Fall back to OLS with HS8 FE and report both |
| Free Comtrade rate limit blocks bulk pull | Use BACI as primary; Comtrade only for spot checks |

---

## Summary for Approval

**What you get:**
1. Three econometric specifications tied one-to-one to Propositions 1, 2, and 5.
2. A fully reproducible data pipeline under `empirical/` with 5 STATA do-files and a documented data layer.
3. A rewritten §3 Empirical Motivation with three new results tables and appendix B.
4. Point estimate of $\hat\sigma$ that replaces the calibrated $\sigma=0.35$ in §5.

**What I will NOT do without further approval:**
- Structural estimation of the full three-player game.
- Formal structural-break test on aggregate $\lambda$.
- Construction of a new causal identification design for Entity List additions.

**Before I execute:** please confirm (a) the Swing State country list is acceptable; (b) BACI (free) is preferred over a premium Comtrade pull; (c) you want §5 recalibrated once $\hat\sigma$ is estimated; (d) you have the 4 person-days of compute-time available.
