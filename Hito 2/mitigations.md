# Mitigations & Deployment Safeguards — Hito 2

**Team:** Feligma  
**Date:** May 13, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  

---

## Executive Summary

This document catalogues concrete failure modes identified in Hito 2 error analysis, assigns severity, and proposes realistic mitigations. The dual-target model is reliable for **midfield constructor strategy decisions** but exhibits critical weaknesses for front-tier and backmarker predictions. Deployment safeguards are required to prevent misuse in high-risk contexts.

---

## Risk Register

### 🔴 **CRITICAL: Backmarker Podium Predictions (is_top3)**

**Failure Mode**
- Test set ROC-AUC = 0.4952 (worse than random chance)
- Positive rate = 0.64% (podium finishes extremely rare)
- Model cannot distinguish patterns in backmarker podium outcomes
- **Consequence:** Recommending strategy optimizations for backmarker drivers to improve P(podium) will be misleading

**Root Cause**
- Confounding: strategy choice for backmarker cars is driven entirely by pace constraints, not by strategic nuance
- Backmarker cars finish podium so rarely that strategy variations are invisible to the model
- Training data (2019–2022) has insufficient positive examples

**Affected Context**
- Constructor tier = backmarker (e.g., Haas, Alfa Romeo, Williams in poor seasons)
- Target = is_top3
- Any what-if scenario involving backmarker podium predictions

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **DO NOT generate is_top3 predictions for backmarker drivers** | In deployment, flag any is_top3 prediction request for backmarker teams with a hard stop: "Podium outcomes too rare; deploy is_top10 model only." | Strategy desk SOP / Model API guard |
| **Use is_top10 instead for backmarker strategy** | Deploy is_top10 (ROC-AUC 0.7925 for backmarker) when optimizing backmarker strategy. Points threshold (top-10) is more achievable than podium. | Strategy advisor documentation |
| **Collect more backmarker podium data (pre-deployment)** | If deployment is scheduled for future season, collect practice session telemetry (FP1, FP2 lap times) and weather forecasts to improve discrimination. | Data engineering team |

**Acceptance Criteria**
- [ ] Deployment API rejects is_top3 predictions for backmarker drivers with human-readable error
- [ ] Strategy desk SOP updated: "For backmarker cars, consult is_top10 model only. is_top3 reliability unknown."
- [ ] Hito 2 report flags this as a critical limitation in public-facing materials

---

### 🔴 **CRITICAL: Front-Tier is_top10 Strategy Discrimination**

**Failure Mode**
- Test set ROC-AUC = 0.6155 (near chance level for is_top10)
- Top teams finish top-10 87.6% of the time
- Model has almost no discriminative power among scenarios
- **Consequence:** Strategy recommendations for front-tier teams to optimize P(top10) will have negligible impact (strategy choices among top teams are dominated by inherent pace, not by tactical variation)

**Root Cause**
- Confounding: top teams finish top-10 so frequently that strategy choice has minimal effect on outcome
- Feature set includes pace proxies (driver_prior3_avg_finish, constructor_prior3_avg_finish) but these capture expected performance, not fine-grained pace discrimination
- Training data conflates "good strategy" with "good car"; separation is impossible without randomized experiment or instrumental variables

**Affected Context**
- Constructor tier = front (e.g., Ferrari, Mercedes, Red Bull)
- Target = is_top10
- Any what-if scenario involving front-tier is_top10 strategy optimization

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Deploy is_top3 for front-tier strategy optimization instead** | For top teams, strategy matters more for podium finishes than for top-10 (P(top10) is nearly 100% anyway). Suggest using is_top3 model (ROC-AUC 0.7872) to optimize for podium. | Strategy advisor SOP |
| **Acknowledge low discriminative power** | In any what-if report involving front-tier is_top10, include a caveat: "Top teams finish top-10 87.6% of the time; strategy has minimal effect on this outcome. This recommendation is low-confidence." | Report generation template |
| **Require pre-race telemetry validation** | Before race day, validate that the strategy desk's pace estimates (from practice sessions FP1, FP2, FP3) align with model expectations. If predicted P(top10) is >95%, strategy choice is irrelevant; signal this to leadership. | Strategy desk + analyst review |

**Acceptance Criteria**
- [ ] Hito 2 report explicitly flags front-tier is_top10 ROC-AUC 0.6155 as "near-random discrimination"
- [ ] Deployment documentation includes: "For front-tier constructors, prioritize is_top3 over is_top10 for strategy decisions"
- [ ] Template for what-if reports includes the caveat above

---

### 🟡 **HIGH: Strategy-Confounding Limitation (Both Targets)**

**Failure Mode**
- Strategy choice in training data (2019–2022) is not independent of car pace, driver ability, weather
- Model learns correlation (strategy ↔ pace), not causation
- When a what-if scenario varies strategy while holding context fixed, we are **not** simulating a causal intervention; we are observing a counterfactual under the training distribution
- **Consequence:** Treating model output as "strategy X causes outcome Y" (causal claim) will lead to overconfident recommendations

**Root Cause**
- Observational data: no randomized control
- Strategy desk makes optimal decisions based on real-time information (practice data, weather, car setup); these decisions are correlated with unmeasured variables (setup quality, tire management expertise, risk appetite)

**Affected Context**
- All what-if scenarios and strategy comparisons
- Both is_top10 and is_top3 targets

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Use scenario-conditioned language consistently** | Every what-if output must include: "Under our training distribution, holding driver-race context fixed, **if we vary only these strategy inputs**, the model predicts..." | Model output template / SOP |
| **Do NOT make causal claims** | Reject any claim of the form: "Strategy A **causes** outcome Y." Rephrase as: "Strategy A is **associated with** outcome Y in our training data." | Report validation checklist |
| **Require subject-matter expert review** | Before deploying a strategy recommendation, have a senior engineer review: "Is this counterfactual realistic given what we know about pace confounding?" | Strategy desk review gate |
| **Include confidence intervals with scenario width** | Report the what-if difference, but also report how much uncertainty attaches to it due to confounding (e.g., "P(top3) delta = 0.10 ± 0.08, 90% CI"). | What-if output format |

**Acceptance Criteria**
- [ ] All what-if comparisons in the report use "scenario-conditioned" language
- [ ] No causal language ("causes," "leads to," "drives") appears in strategy recommendations
- [ ] Hito 2 report includes a section: "Why We Don't Claim Causation" explaining strategy confounding

---

### 🟡 **HIGH: Calibration Degradation by Constructor Tier**

**Failure Mode**
- Model calibration varies significantly by constructor tier
- Front tier is_top10: Brier 0.1085 (good) but ROC-AUC 0.6155 (poor discrimination)
- Backmarker tier is_top3: Brier 0.0064 (suspiciously low), ROC-AUC 0.4952 (worse than random)
- **Consequence:** Predicted probabilities may be poorly calibrated for edge cases (front/backmarker extreme tiers)

**Root Cause**
- Training data imbalance: front teams vastly overrepresented among top-10 finishes; backmarker teams vastly underrepresented among podiums
- Isotonic calibration (fit on 2022 data) may not generalize to extreme subgroups

**Affected Context**
- Constructor tier = front or backmarker (not midfield)
- Any prediction for a constructor outside the "normal" range observed in calibration set

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Stratified model option** | For final deployment, consider training separate models for each constructor tier (front / midfield / backmarker) rather than a single global model. This removes confounding within tier. | Data science team (future work) |
| **Plausibility check on probabilities** | When model outputs P(top10) > 0.95 for any front-tier driver, flag it as "probability is high due to class imbalance; treat as ≈certainty, not a precise point estimate." | Output validation |
| **Recalibrate on recent season data** | Before each season, refit Isotonic calibration on the previous season (e.g., recalibrate 2023 on 2022; recalibrate 2024 on 2023) to reduce distribution shift. | Model maintenance SOP |

**Acceptance Criteria**
- [ ] Hito 2 report includes a subsection: "Calibration Quality by Constructor Tier" with metrics
- [ ] Any deployment includes a note: "Model calibration varies by team tier; see calibration curves in appendix"
- [ ] Strategy desk training includes guidance on interpreting probabilities for edge-case teams

---

### 🟠 **MEDIUM: Feature Leakage Risk from Scenario Inputs**

**Failure Mode**
- `n_stops` and `compound_sequence` are classified as "scenario inputs" (user-varied for what-if)
- However, in the training data, these variables are **observed post-race**
- If the model has learned spurious correlations to post-race features, then what-if scenarios may not be valid
- **Consequence:** Scenario-based recommendations might reflect overfitting to post-race patterns rather than true strategy effects

**Root Cause**
- Design choice: treating post-race strategy variables as "scenario inputs" requires careful justification
- Validation: We assume strategy inputs have no post-race information leakage, but this is hard to verify

**Affected Context**
- All what-if scenarios (Step 4 of notebook)
- Both targets (is_top10 and is_top3)

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Validate scenario input independence** | Confirm that `n_stops` and `compound_sequence` do not appear in feature set alongside post-race race-condition variables. | Leakage audit (Step 2 of notebook) ✅ Already done |
| **Compare post-hoc vs model-based predictions** | For a held-out test race, compare the model's scenario-based is_top10 predictions to the actual historical strategy-outcome pairs from that race. If predictions align with historical patterns, scenario inputs are likely valid. | Validation experiment (future) |
| **Interpret scenarios as "what would have happened if" not "what will happen if"** | Remind strategy desk: scenarios show historical counterfactuals (e.g., "if Sainz had done 1-stop instead of 2-stop at Monza 2024, model predicts..."), not forward-looking forecasts of future races. | SOP language |

**Acceptance Criteria**
- [ ] Leakage audit (leakage_audit.md) confirms no post-race leakage for scenario inputs ✅
- [ ] Hito 2 report includes: "Scenario Input Validation" section
- [ ] Documentation clarifies: scenarios are counterfactuals on training distribution, not forward predictions

---

### 🟠 **MEDIUM: Limited Predictive Horizon**

**Failure Mode**
- Model is trained on 2019–2021 (train) and calibrated on 2022 (calib), tested on 2023–2024
- Distributional shift across seasons: car regulations changed (2022 new regs), tire suppliers vary, circuit modifications
- Performance on 2025+ seasons unknown
- **Consequence:** Model predictions degrade over time; recommendations from mid-2026 may not apply to late-2026 season

**Root Cause**
- All ML models suffer from dataset shift over time
- F1 introduces rule changes mid-season, mid-regulation-cycle; strategy implications change accordingly

**Affected Context**
- Predictions beyond 2024 test set
- Any race with significant regulation change (tire compounds, pit stop procedures)

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Retrain model quarterly** | After each racing phase (e.g., post-summer break), retrain on the latest 3-year window (e.g., in July 2026, retrain on 2023–2025). | Model maintenance SOP |
| **Monitor performance drift** | For each race in deployment, compute Brier score and log it. If Brier > threshold (e.g., 0.15 for is_top10), flag for retraining. | Monitoring dashboard |
| **Invalidate predictions for major rule changes** | If FTC announces mid-season regulation change (tire compounds, pit procedures), halt model predictions until retraining completes. | Strategy desk alert protocol |

**Acceptance Criteria**
- [ ] Hito 2 report includes: "Model Lifespan & Retraining Schedule"
- [ ] Deployment includes a retraining checklist with quarterly cadence
- [ ] Monitoring dashboard available to strategy team showing performance drift

---

### 🟢 **LOW: Data Sparsity in Edge Cases**

**Failure Mode**
- Certain combinations of (constructor_tier, n_stops, circuit_type) are rare in training data
- For example: backmarker drivers on semi-street circuits with 3+ stops
- Model makes predictions on sparse regions of feature space
- **Consequence:** Predictions in edge cases have high variance / low reliability

**Root Cause**
- F1 has only ~23 races per season; training set (2019–2021) = ~69 races
- Many (tier × strategy × circuit) combinations have <10 examples

**Affected Context**
- Scenarios involving rare constructor-circuit-strategy combinations
- Particularly: backmarker + semi-street circuits

**Mitigation(s)**

| Mitigation | Implementation | Responsibility |
|---|---|---|
| **Flag low-data scenarios** | When generating a what-if report, count how many training examples match the scenario's (tier × strategy × circuit) signature. If < 10, flag as "low-data edge case." | What-if validation function |
| **Use prediction intervals** | For Hito 2, report point prediction + uncertainty band (e.g., 90% prediction interval from bootstrap of base_model training) instead of point estimate. | Output format |
| **Request expert judgment** | If a scenario is in an edge case, require strategy staff to review before acting on the recommendation. | Decision gate |

**Acceptance Criteria**
- [ ] What-if template includes a "Data Availability" row showing training N for the scenario's slice
- [ ] Hito 2 report explains how low-data scenarios are handled

---

## Summary: Where Is This Model Reliable?

### ✅ **GREEN ZONE: Midfield Constructor Strategy Decisions**

**Profile**
- Constructor tier = midfield (e.g., Alpine, Aston Martin in typical seasons)
- Target = is_top10 or is_top3
- Circuit type = any
- Strategy variation = 1-stop vs 2-stop (common choices in training data)

**Performance**
- is_top10: ROC-AUC 0.8315, Brier 0.1593 → Good discrimination
- is_top3: ROC-AUC 0.8876, Brier 0.0914 → Excellent discrimination

**Recommendation**
- ✅ **Safe to deploy** for strategy staff strategy decisions in midfield context
- Requires: scenario-conditioned language, subject-matter expert review, monitoring

---

### ⚠️ **YELLOW ZONE: Front-Tier Podium Optimization (is_top3)**

**Profile**
- Constructor tier = front
- Target = is_top3 (NOT is_top10)
- Strategy focus = aggressive vs conservative compounds / pit window

**Performance**
- is_top3: ROC-AUC 0.7872, Brier 0.1901 → Moderate-good discrimination

**Recommendation**
- ⚠️ **Deploy with caution** for front-tier podium strategy
- Requires: explicit caveat that is_top10 discrimination is poor (teams finish top-10 too often)
- Requires: pre-race telemetry validation (FP3 pace estimates)

---

### 🔴 **RED ZONE: Do Not Deploy**

**Backmarker is_top3 (Podium)**
- Constructor tier = backmarker
- Target = is_top3

**Performance**
- ROC-AUC 0.4952 (worse than random), Brier 0.0064 (unreliable)

**Recommendation**
- ❌ **Do not deploy** for backmarker podium predictions
- Use is_top10 instead (ROC-AUC 0.7925 for backmarker), or collect more data

---

## Mitigations Checklist for Hito 2

- [x] Strategy-confounding limitation identified and documented (leakage_audit.md)
- [x] Front-tier is_top10 weakness flagged (ROC-AUC 0.6155)
- [x] Backmarker is_top3 weakness flagged (ROC-AUC 0.4952)
- [x] Scenario-conditioned language defined (Section on confounding mitigation)
- [x] Calibration concerns by tier documented
- [x] Feature leakage risk assessed (valid: n_stops, compound_sequence have no post-race leakage)
- [x] Retraining schedule proposed (quarterly cadence)
- [x] Deployment safeguards proposed (hard stops for red-zone scenarios)
- [ ] Strategy desk training materials created (future: prepare SOP for operations)
- [ ] Monitoring dashboard deployed (future: build alerting system)

---

## Mitigations Checklist for Final Report & Deployment

Before deployment in real F1 season:

- [ ] Retrain model on 2019–2024 data (latest available)
- [ ] Recalibrate Isotonic scaler on most recent calibration season (2024)
- [ ] Stratified calibration curves by constructor tier (plot Brier & calibration curves for front/mid/back)
- [ ] Deploy deployment API with hard stops for red-zone scenarios
- [ ] Prepare strategy desk SOP document: "How to Use the Model" with examples
- [ ] Set up monitoring dashboard: Brier score tracker, alert if drift > threshold
- [ ] Prepare user education: why scenario-conditioned language matters
- [ ] Test end-to-end on a historical race (e.g., re-score a 2024 race with 2023 model as if deployed)

---

## References

- **Error Analysis Details:** See [leakage_audit.md](leakage_audit.md#critical-confounding-signal-by-constructor-tier)
- **Model Performance:** See [baseline_comparison.md](baseline_comparison.md)
- **Leakage Audit:** See [leakage_audit.md](leakage_audit.md)
- **Notebook Implementation:** See `hito2_modeling.ipynb`, Section 0 (Hito 1 corrections) and Steps 3–4 (dual-target training)
