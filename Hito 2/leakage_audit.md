# Leakage & Confounding Audit — Hito 2

**Team:** Feligma  
**Date:** May 13, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  
**Notebook:** `hito2_modeling.ipynb` Step 2 (Leakage Guard)

---

## Executive Summary

This document audits data leakage and confounding in the dual-target framework. The core finding: **strategy features are correctly classified as scenario inputs and do not leak future information**. The model uses a curated set of **7 features** (5 pre-race context + 2 strategy scenario inputs) and applies a locked temporal split (2019–2021 train / 2022 calibration / 2023–2024 test). However, a critical **strategy-confounding limitation** exists: strategy choice in the training data (2019–2022) is not independent of car pace, driver skill, and weather. This affects both targets and is mitigated via scenario-conditioned interpretation.

---

## Column Classification & Leakage Guard

The notebook's Step 2 (leakage guard) classifies all dataset columns into five categories:

### ✅ Pre-Race — Safe for model fit (5 used in CORRECTED_FEATURES)

| Column | Class | In CORRECTED_FEATURES? | Justification | Status |
|---|---|---|---|---|
| `grid_position` | pre_race | ✅ YES | Known before race start; set by qualifying | ✅ Used in model |
| `driver_prior3_avg_finish` | pre_race | ✅ YES | 3-race historical average; known before this race | ✅ Used in model |
| `constructor_prior3_avg_finish` | pre_race | ✅ YES | 3-race historical average; known before this race | ✅ Used in model |
| `driver_circuit_prior_avg` | pre_race | ✅ YES | Historical average at this circuit; known before race | ✅ Used in model |
| `constructor_tier` | pre_race | ✅ YES | Known from team roster; does not change mid-race | ✅ Used in model |
| `circuit_type` | pre_race | ❌ NO | Used only for error-analysis segmentation | ⚠️ Not in model (segmentation only) |
| `qualifying_position` | pre_race | ❌ NO | Set before the race; **excluded from CORRECTED_FEATURES** | ⚠️ Not used (collinear with grid_position; Hito 1 correction) |

> **CORRECTED_FEATURES composition (7 features total):**
>   - **Pre-race context (5):** grid_position, driver_prior3_avg_finish, constructor_prior3_avg_finish, driver_circuit_prior_avg, constructor_tier
>   - **Strategy scenario inputs (2):** n_stops, compound_sequence
>   - **Excluded from model but used for analysis segmentation:** circuit_type, weather_actual, safety_car_laps (classified as pre_race/audit but kept out of model fit)

> **Hito 1 corrections carried into Hito 2:** `round` was dropped (race identifier, not predictive); `avg_track_temp` and `avg_air_temp` were dropped (post-race observations); `qualifying_position` was dropped (collinear with grid_position). None of these appear in `CORRECTED_FEATURES`.

**Conclusion:** All 7 features used in `CORRECTED_FEATURES` are legitimately available before the strategy decision is made. No leakage in this set.

---

### ✅ Scenario Input — Strategy control variables

| Column | Class | In CORRECTED_FEATURES? | Varied in What-If? | Status |
|---|---|---|---|---|
| `n_stops` | scenario_input | ✅ YES | ✅ YES | ✅ Scenario varied |
| `compound_sequence` | scenario_input | ✅ YES | ✅ YES | ✅ Scenario varied |
| `strategy_type` | scenario_input | ❌ No (redundant with n_stops) | ❌ No | ⚠️ Not in model |
| `stint1_length`, `stint2_length`, `stint3_length` | scenario_input | ❌ No | ❌ No | ⚠️ Observed post-race; not in model |
| `avg_pit_stop_duration_s`, `total_pit_time_s` | scenario_input | ❌ No | ❌ No | ⚠️ Observed post-race; not in model |
| `first_pit_lap`, `last_pit_lap` | scenario_input | ❌ No | ❌ No | ⚠️ Observed post-race; not in model |
| `stint4_length`, `stint5_length` | scenario_input | ❌ No | ❌ No | ⚠️ Observed post-race; not in model |

> **Scope change vs Hito 1:** The Hito 1 framing varied `n_stops`, `compound_sequence`, and `avg_pit_stop_duration_s`. For Hito 2 we **restricted the scenario surface to just `n_stops` and `compound_sequence`** — the two strategy choices the desk fixes pre-race. Stint lengths and pit durations are observed post-race in this dataset; including them as scenario inputs would conflate strategic intent with the realised race execution. This is the cleanest defensible scenario_input set given the column semantics.

**Protocol:** In what-if scenarios (Step 4 of the notebook), only `n_stops` and `compound_sequence` are varied. All other features remain at the base row's values. The `score_pair()` function enforces this constraint at runtime — it raises a `ValueError` if any key outside `SCENARIO_INPUT_COLS` is passed in a scenario dict. This satisfies the leakage constraint: we are not using post-race pit duration or realised stint lengths to inform the what-if prediction.

**Conclusion:** Scenario inputs are correctly classified. What-if comparisons are leakage-free.

---

### ⚠️ Audit — Race conditions, used for segmentation only

| Column | Class | Justification | In Model Fit? | Used in Slicing? |
|---|---|---|---|---|
| `safety_car_laps`, `safety_car_periods`, `vsc_laps` | audit | Race incidents; observed during race | ❌ No | ✅ Could segment (future work) |
| `weather_actual`, `avg_air_temp`, `avg_track_temp` | audit | Observed during race | ❌ No | ✅ Could segment (future work) |
| `wet_laps` | audit | Observed during race | ❌ No | ✅ Could segment (future work) |
| `track_status_summary` | audit | Observed during race | ❌ No | ✅ Could segment (future work) |

**Conclusion:** These columns are correctly excluded from model fit and are reserved for error-analysis segmentation.

---

### ❌ Outcome — Labels and results, excluded from fit

| Column | Class | Status |
|---|---|---|
| `is_top10`, `is_top3`, `is_top5` | outcome | ❌ Not used as features |
| `finish_position`, `points` | outcome | ❌ Not used as features |
| `dnf`, `positions_gained` | outcome | ❌ Not used as features |
| `status` | outcome | ❌ Not used as features |

**Conclusion:** All outcome columns correctly excluded from `CORRECTED_FEATURES`.

---

### 🔍 Identifiers — Not used in model

| Column | Class | Treatment |
|---|---|---|
| `Driver`, `Team`, `circuit`, `season` | id | Index/filter only |
| `circuit_id`, `driver_id`, `driver_name`, `constructor_name`, `race_name` | id | Not used |
| `round` | id | Dropped in Hito 1 corrections |
| `stint_lengths` | unclassified | Redundant with `stint1_length`, etc. |

**Conclusion:** Identifier columns correctly excluded from features.

---

## Leakage Verdict: ✅ CLEAN

| Dimension | Status | Evidence |
|---|---|---|
| Temporal split respected | ✅ Pass | Train 2019–2021 / Calib 2022 / Test 2023–2024 — no data leakage across splits |
| Target features excluded | ✅ Pass | `is_top10`, `is_top3`, `finish_position`, `points` not in `CORRECTED_FEATURES` |
| Post-race features not in fit | ✅ Pass | Audit columns (weather, safety car, pit duration) not in model fit |
| Scenario inputs valid | ✅ Pass | Only `n_stops` and `compound_sequence` varied in what-if; all others held constant by `score_pair()` |
| Pre-race feature set complete | ✅ Pass | All features in the model are known before race start or are strategy control variables |
| Calibration procedure consistent | ✅ Pass | Both targets use `LogisticRegression` (fit on 2019–2021) + Isotonic calibration via `FrozenEstimator` (fit on 2022). No model selection on test set. |

**The dual-target model is leakage-free on both `is_top10` and `is_top3`.**

---

## Confounding Limitation: Strategy Is Not Random

### The Problem

In the training data (2019–2022), strategy choice is **not independent** of:

1. **Car pace.** Teams with faster cars are more likely to choose two-stop aggressive strategies. Teams with slower cars are more likely to choose one-stop conservative strategies.

2. **Driver skill.** Championship-contending drivers (fast teams, proven drivers) are assigned more aggressive strategies more often. Lower-tier drivers get more conservative calls.

3. **Weather and track conditions.** Safety-car deployments, wet patches, and tyre-degradation profiles are known to the strategy desk at decision time. Strategy is chosen *conditional on* these observations.

4. **Team philosophy.** Ferrari, Mercedes, and Red Bull have different strategic playbooks and risk appetites.

**Consequence:** When we observe "two-stop achieves higher P(top3) in the training data," we are observing both:
- **(A) The causal effect of the strategy itself**, and
- **(B) The confounding: teams with good pace use two-stop, and good-pace teams finish higher.**

Our model **cannot separate (A) from (B).**

### Manifestation in Both Targets

Both `is_top10` and `is_top3` are affected by this confounding. Evidence from the homogenised test-set error analysis (2023–2024):

| Target | Strategy Comparison | Confounding Effect |
|---|---|---|
| **is_top10** | 1-stop Brier 0.1473 vs 2-stop Brier 0.1256 | 2-stop performs better, but two-stop teams were systematically faster in training |
| **is_top3** | 1-stop Brier 0.0861 vs 2-stop Brier 0.0809 | 2-stop outperforms, but higher-pace teams selected 2-stop more frequently |
| **Constructor Tier** | Front teams ROC-AUC 0.6878 (is_top10); Backmarker ROC-AUC 0.4952 (is_top3) | Strategy effects are confounded with inherent team pace/capability |

---

### Why This Matters for Interpretation

#### ❌ Causal Claim (INVALID)

> "Two-stop **causes** higher podium probability."

This is **not valid** because strategy choice is confounded with pace.

#### ✅ Scenario-Conditioned Claim (VALID)

> "Under our training distribution, **holding driver-race context fixed** (including pace proxies like `driver_prior3_avg_finish` and `constructor_prior3_avg_finish`), **if we vary only the strategy inputs**, the model predicts P(top3) increases with two-stop."

This is valid because we're holding the confounding variables fixed.

---

### How We Mitigate This Confounding

1. **Scenario-conditional language.** Every what-if interpretation in Hito 2 uses the phrase "under our training distribution, holding context fixed." This reminds the reader we are not making causal claims.

2. **Pace proxies in feature set.** The model includes `driver_prior3_avg_finish` and `constructor_prior3_avg_finish`, which capture pace indirectly. When we hold these fixed and vary strategy, we are partially controlling for pace confounding.

3. **Explicit limitation discussion.** Both this document and `mitigations.md` highlight that strategy confounding is a limitation and explain what pre-race telemetry and practice data the strategy desk would need to decompose it.

---

## Implications for Both Targets

### is_top10: Performance Varies by Constructor Tier

For `is_top10`, performance varies significantly by constructor tier (homogenised metrics):

| Constructor Tier | Brier | ROC-AUC | Issue |
|---|---|---|---|
| **Front** | 0.1071 | 0.6878 | ⚠️ Weak ROC-AUC; model is near-random for top-tier teams |
| **Midfield** | 0.1581 | 0.8344 | Reasonable performance; strategy features add moderate value |
| **Backmarker** | 0.1270 | 0.7693 | Workable for tactical top-10 advice |

**Confounding interpretation:** The weak ROC-AUC for front teams indicates that strategy choice is highly confounded with underlying pace. Front teams finish top-10 so frequently (87.6% positive rate) that strategy variations have minimal effect on top-10 outcomes — strategy decisions for front teams are about *podium*, not top-10.

### is_top3: Backmarker Tier Is Unreliable

For `is_top3`, the confounding is extreme in the backmarker tier:

| Constructor Tier | Brier | ROC-AUC | Issue |
|---|---|---|---|
| **Front** | 0.1901 | 0.7872 | Brier is high; podium predictions are uncertain for top teams |
| **Midfield** | 0.0914 | 0.8876 | Reasonable performance; strategy has interpretable effect |
| **Backmarker** | 0.0064 | 0.4952 | ⚠️ **Backmarker podium predictions are unreliable.** Positive rate = 0.64%; ROC-AUC ≈ random |

**Confounding interpretation:** The extremely low positive rate (0.64%) for backmarker teams means podium finishes are so rare that no strategy variation appears predictive. The strategy desk never recommends an aggressive strategy for backmarker cars because pace is insufficient. Our model cannot distinguish between "strategy doesn't matter" and "confounding between strategy and pace is total."

**Mitigation:** What-if scenarios involving backmarker drivers should include explicit caveats that podium predictions are unreliable and strategy recommendations are only valid for competitive teams (front/midfield tiers).

---

## Confounding Checklist

- [x] **Identified:** Strategy choice correlates with car pace, driver, weather
- [x] **Documented:** This section explains the mechanism
- [x] **Mitigated in interpretation:** All what-if claims use scenario-conditioned language
- [x] **Mitigated in feature set:** Pace proxies included in model
- [x] **Mitigated in mitigations.md:** Deployment safeguards documented separately
- [x] **Both targets affected:** Analysis applies to is_top10 and is_top3

---

## Key Findings from Error Analysis (Hito 2 Notebook)

The tables below are reproduced from `hito2_modeling.ipynb` Step 4b under the homogenised pipeline (Isotonic calibration via `FrozenEstimator`, identical procedure for both targets). Run the notebook end-to-end to regenerate; the numbers below match a clean run as of May 13, 2026.

### By Strategy Type (n_stops)

| Strategy | N | is_top10 Brier | is_top10 ROC-AUC | is_top3 Brier | is_top3 ROC-AUC |
|---|---|---|---|---|---|
| 1-stop | 353 | 0.1473 | 0.8605 | 0.0861 | 0.9002 |
| **2-stop** | 368 | **0.1256** | **0.9011** | **0.0809** | **0.9284** |
| 3+-stop | 153 | 0.1509 | 0.8638 | 0.0720 | 0.8588 |
| no-stop | 15 | 0.0610 | NaN | 0.0220 | NaN |

**Confounding interpretation:** 2-stop outperforms 1-stop on Brier for both is_top10 (0.1256 < 0.1473) and is_top3 (0.0809 < 0.0861), **but this does not prove 2-stop is a better strategy**. Teams that chose 2-stop in training data were systematically more competitive. The feature set includes pace proxies that partially capture this, but separation is incomplete.

### By Circuit Type

| Circuit Type | N | is_top10 Brier | is_top10 ROC-AUC | is_top3 Brier | is_top3 ROC-AUC |
|---|---|---|---|---|---|
| Permanent | 579 | 0.1320 | 0.8888 | 0.0834 | 0.8942 |
| Semi-street | 118 | 0.1727 | 0.8272 | 0.0676 | **0.9486** |
| Street | 192 | 0.1323 | 0.8855 | 0.0795 | 0.9242 |

**Finding:** Semi-street circuits show a confounding pattern — worse on is_top10 (Brier 0.1727), excellent on is_top3 (ROC-AUC 0.9486). Semi-street races have specific conditions (higher within-race variance, safety cars) where only top-tier teams reliably reach podium, making is_top3 easier to predict at the cost of harder is_top10 discrimination.

### By Constructor Tier — Critical Confounding Signal

Constructor tier reveals the strongest confounding signal:

| Tier | N | is_top10 Brier | is_top10 ROC-AUC | is_top3 Brier | is_top3 ROC-AUC | Notes |
|---|---|---|---|---|---|---|
| **Front** | 170 | 0.1071 | **0.6878** ⚠️ | 0.1901 | 0.7872 | Top teams finish top-10 87.6% of the time; strategy is dominated by pace; podium predictions uncertain |
| **Midfield** | 407 | 0.1581 | 0.8344 | 0.0914 | 0.8876 | Strategy decisions matter here; deployment-safe |
| **Backmarker** | 312 | 0.1270 | 0.7693 | **0.0064** | **0.4952** ⚠️ | Podium finishes extremely rare (0.64%); ROC-AUC ≈ random; is_top3 unreliable |

**Critical interpretation:** The front-tier ROC-AUC of 0.6878 for is_top10 is near chance, indicating the model has weak discriminative power for top teams on the top-10 boundary. This is **pure confounding**: top teams' inherent pace dominates outcomes, making strategy effects on top-10 nearly invisible. The backmarker ROC-AUC of 0.4952 for is_top3 indicates the model cannot distinguish podium-relevant patterns because podium outcomes are too rare.

---

## Appendix: What Would Fix This?

To remove strategy confounding entirely, we would need:

1. **A/B testing:** Some races with forced strategy assignment (strategy desk assigns strategies randomly, not optimally). No F1 team would accept this.

2. **Causal structure learning:** Fit a causal model with strategy choice as an intermediate node, using instrumental variables or latent-confounder approaches. Requires more data and careful modelling.

3. **Stratified training:** Fit separate models for each pace tier (top teams vs midfield vs backmarker), each with their own strategy-outcome relationship. Feasible, but reduces training sample size and generalisation.

4. **Pre-race telemetry:** Include practice-session pace data (FP1, FP2, FP3 lap times) as additional pre-race features. This would allow the model to infer pace more accurately and reduce reliance on strategy features to capture it.

For Hito 2 we mitigate confounding via scenario-conditioned interpretation and explicit limitation documentation. For the final report and deployment, option 4 (pre-race telemetry) is the most realistic.

---

## Conclusion

The dual-target model (`is_top10` and `is_top3`) is **leakage-free** in terms of temporal separation and feature classification. Both targets are affected by a **strategy-confounding limitation** inherent to the training data, not a modelling error.

### Severity Assessment

| Component | Severity | Impact |
|---|---|---|
| **is_top10 (Front tier)** | 🔴 **Critical** | ROC-AUC 0.6878 is near-random; strategy effects are dominated by inherent pace |
| **is_top3 (Backmarker tier)** | 🔴 **Critical** | ROC-AUC 0.4952 is random; podium predictions are uninformative |
| **is_top10 (Midfield tier)** | 🟡 **Moderate** | ROC-AUC 0.8344; strategy has interpretable effects, confounding with pace remains |
| **is_top3 (Midfield tier)** | 🟢 **Manageable** | ROC-AUC 0.8876; good discriminative power for strategy-informed decisions |

### Recommended Model Application

- ✅ **Safe:** Strategy recommendations for midfield teams (both is_top10 and is_top3)
- ⚠️ **Caveat:** Strategy recommendations for front teams on is_top10 (inherent pace dominates) — prefer is_top3 for front-tier strategy
- ❌ **Do not use:** Podium (is_top3) predictions for backmarker drivers (essentially uninformative)

**This limitation is documented and addressed in:**
- Scenario-conditioned interpretation (all what-if claims acknowledge training distribution)
- Pace proxies in the feature set (`driver_prior3_avg_finish`, `constructor_prior3_avg_finish`)
- Explicit discussion in `mitigations.md`
- Deployment safeguards: require real-time telemetry (practice-session lap times, tyre data) before race-day strategy finalisation
