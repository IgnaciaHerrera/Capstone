# Error Analysis — Hito 2 Dimension 1

**Objective:** Identify where our dual-target F1 race strategy models struggle and whether they exhibit systematic disagreement. Analysis slices the 2023–2024 test set by **strategy type** (n_stops: no-stop/1-stop/2-stop/3+-stop), **circuit type** (street / permanent / semi-street), and **constructor tier** (front / midfield / backmarker). Metrics: Brier score and ROC-AUC for both `is_top10` (primary) and `is_top3` (expansion). All numbers below are reproduced from `hito2_modeling.ipynb` Step 4b under the homogenised pipeline (LogisticRegression + Isotonic calibration via `FrozenEstimator`, same procedure for both targets).

---

## 1. Strategy Type (n_stops)

| Strategy   | n    | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|-----------|------|----------------|---------------|-------------|---------------|-----------|---------------|
| No-stop   | 15   | 0.0%           | 0.0%          | 0.0610      | NaN           | 0.0220    | NaN          |
| 1-stop    | 353  | 56.1%          | 16.4%         | 0.1473      | 0.8605        | 0.0861    | 0.9002       |
| **2-stop**| 368  | 54.3%          | 17.4%         | **0.1256** ✓ | **0.9011** ✓  | **0.0809**✓ | **0.9284**✓ |
| 3+-stop   | 153  | 40.5%          | 10.5%         | 0.1509      | 0.8638        | 0.0720    | 0.8588       |

### Key Findings

1. **The model excels at 2-stop scenarios.** Brier 0.1256 (top10) and 0.0809 (top3) are the best across strategies. 2-stop is the best-represented strategy in training data (368 test cases) and shows the highest ROC-AUC on both targets.

2. **No-stop is rare and uninformative.** Only 15 cases (1.7% of test set) with 0% top-10 and top-3 finishes. ROC-AUC is NaN because there is no positive-class variation. These are edge cases (safety-car deployments, wet-race outliers). The model correctly predicts very low top-10 probabilities, but the slice provides no discrimination signal.

3. **3+-stop is rare and weaker.** Only 153 cases (17.2% of test set). Top-10 prevalence drops to 40.5%. ROC-AUC 0.8638 for top10 is respectable, but the model struggles more than on 1-stop or 2-stop, consistent with 3+-stop being rarer in the 2019–2022 training distribution.

4. **1-stop and 2-stop have similar positive rates (~54–56%) but 2-stop is modelled more reliably.** This suggests either (i) the training data is better-labelled or pit windows are more stable for 2-stop, or (ii) confounding — fast teams in the training period chose 2-stop more often, so 2-stop carries a "good car" signal the model has implicitly learned.

### Implication for Advisor

Users requesting 1-stop strategy evaluations should expect slightly lower model confidence (ROC-AUC 0.8605 vs 0.9011 for 2-stop). 3+-stop scenarios (17% of test) should include a caveat about limited training representation. No-stop recommendations should be flagged as unreliable edge cases requiring human review.

---

## 2. Circuit Type

| Circuit Type  | n   | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|--------------|-----|----------------|---------------|-------------|---------------|-----------|---------------|
| **Permanent** | 579 | 51.8%          | 15.5%         | **0.1320**✓ | **0.8888**✓  | 0.0834    | 0.8942       |
| Semi-street  | 118 | 50.8%          | 15.3%         | 0.1727      | 0.8272        | 0.0676    | **0.9486**✓ |
| Street       | 192 | 52.1%          | 15.6%         | 0.1323      | 0.8855        | 0.0795    | 0.9242       |

### Key Findings

1. **Permanent circuits are strongest for top10.** ROC-AUC 0.8888 for `is_top10`, Brier 0.1320. High-speed, high-pit-delta circuits have stable strategy outcomes; with 579 test cases, the model has the most training/test signal here.

2. **Semi-street is the weakest slice for top10 and the strongest for top3.** Brier 0.1727 / ROC-AUC 0.8272 on top10 (worst), but Brier 0.0676 / ROC-AUC 0.9486 on top3 (best). The likely mechanism: semi-street races have higher within-race variance (track-evolution windows, safety cars) that hurts mid-pack top-10 prediction, but the *podium* contest is more deterministic because only front-tier cars realistically reach P3 on these layouts. Small n (118) means the top3 ROC-AUC should be interpreted with confidence-interval caution.

3. **Street circuits hold up well.** ROC-AUC 0.8855 (top10) and 0.9242 (top3) on 192 cases. Street circuits do *not* underperform overall — earlier hypotheses (Hito 1 framing) about street being especially hard for the model are not supported by these numbers.

### Implication for Advisor

All circuit types show ROC-AUC > 0.82 for top10 and > 0.89 for top3 — the model is reasonably robust across circuit types. Permanent circuits (n=579) are the safest deployment zone. Street circuits (n=192) perform comparably. Semi-street circuits (n=118) deserve a caveat on top10 predictions but the top3 signal is unusually strong, with the small-sample caveat noted.

---

## 3. Constructor Tier (based on prior3_avg_finish)

**Justification for this context:** Constructor tier was selected as the additional context because it is the strongest pre-race predictor of class imbalance. Front-tier teams (Mercedes, Red Bull, Ferrari) have a fundamentally different top-3 base rate (>45%) than midfield (<15%) and backmarkers (<1%), making it the most likely source of systematic model failure under logistic regression. Including this slice directly addresses whether the model's calibration degrades predictably with extreme prevalence shifts.

| Constructor Tier   | n   | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|--------------------|-----|----------------|---------------|-------------|---------------|-----------|---------------|
| **Front (≤5.0)**   | 170 | 87.6%          | 45.3%         | 0.1071      | **0.6878** ⚠️  | 0.1901    | **0.7872** ⚠️ |
| Midfield (5–10)    | 407 | 61.9%          | 14.5%         | 0.1581      | 0.8344        | 0.0914    | 0.8876       |
| **Backmarker (>10)**| 312 | 18.9%          | 0.6%          | 0.1270      | 0.7693        | **0.0064**✓ | **0.4952** ⚠️ |

### Key Findings

1. **Front-tier constructors: discrimination collapses on both targets.**
   - 170 cases, **45.3% are top3 finishers** — extreme class imbalance.
   - ROC-AUC drops to 0.6878 for `is_top10` (near-random) and 0.7872 for `is_top3`.
   - **Root cause:** Logistic regression discrimination weakens when one class dominates. The model is broadly correct on average (low Brier 0.1071 for top10) but cannot reliably *rank* scenarios where almost everyone finishes top-10.
   - **Why Brier on top10 is still low (0.1071):** Predicting near-1 for everyone is accurate on average for a class with 87.6% positive rate, but it is not useful for strategy discrimination.

2. **Midfield is the sweet spot for the model.**
   - 407 cases (largest category, 46% of test set).
   - ROC-AUC 0.8344 (top10) and 0.8876 (top3) on balanced class distributions. This is where strategy advice carries the most decision-value.

3. **Backmarker is_top3 is uninformative; backmarker is_top10 is workable.**
   - 312 cases (35% of test set), but only 0.6% finish top3 — perfectly aligned with the realities of mid-2020s F1.
   - Brier 0.0064 (top3) is near the null baseline for this prevalence (0.006 × 0.994 ≈ 0.006). The model achieves this almost entirely by predicting near-zero probability for all backmarker cases. This is not evidence of model quality.
   - ROC-AUC 0.4952 (top3) is computed on only 2 positive cases out of 312 — statistically uninformative. Effectively random.
   - Backmarker is_top10 is more usable (ROC-AUC 0.7693, Brier 0.1270): the model can distinguish good vs poor backmarker races even if podium is off the table.

### Implication for Advisor

**Deployment risk:** Strategy recommendations for **front-tier constructors should be issued with caveats** — the model's top10 discrimination is near-random (ROC-AUC 0.6878) and top3 discrimination is moderate (0.7872). Practical reading: front teams finish top-10 too frequently for strategy choice to materially change the top-10 outcome; their meaningful strategy decisions are about *podium* and *win*, not top-10. **For backmarker teams**, *do not use* the `is_top3` model (ROC-AUC 0.4952). Backmarker `is_top10` predictions are usable as a tactical aid.

---

## 4. Target Disagreement Patterns

**Do `is_top10` and `is_top3` predictions align, or do they diverge?**

### By Strategy

- **No-stop:** Both targets have NaN/undefined ROC-AUC (no positive cases). No meaningful comparison.
- **1-stop:** ROC-AUC Top10 = 0.8605 vs Top3 = 0.9002 → **targets AGREE** (both strong, gap 0.040). One-stop is well-modelled for both targets.
- **2-stop:** ROC-AUC Top10 = 0.9011 vs Top3 = 0.9284 → **targets AGREE** (both very strong, gap 0.027). Two-stop is the model's strongest slice.
- **3+-stop:** ROC-AUC Top10 = 0.8638 vs Top3 = 0.8588 → **targets AGREE** (both moderate, gap 0.005).

### By Circuit

- **Permanent circuits:** ROC-AUC Top10 = 0.8888 vs Top3 = 0.8942 → **targets AGREE** (both excellent, gap 0.005).
- **Semi-street circuits:** ROC-AUC Top10 = 0.8272 vs Top3 = 0.9486 → **targets DISAGREE** (Top3 is +0.121 better). Mixed terrain favours podium ranking over top10.
- **Street circuits:** ROC-AUC Top10 = 0.8855 vs Top3 = 0.9242 → **targets DISAGREE** (Top3 is +0.039 better). Street circuits show a slight top3 advantage.

### By Constructor

- **Front teams:** ROC-AUC Top10 = 0.6878 vs Top3 = 0.7872 → **targets DISAGREE** (both weak, but Top3 is +0.099 stronger). Extreme class imbalance breaks both models; Top3 is the less broken of the two.
- **Midfield teams:** ROC-AUC Top10 = 0.8344 vs Top3 = 0.8876 → **targets AGREE** (both strong, gap 0.053). Balanced class distribution lets both targets perform well.
- **Backmarkers:** ROC-AUC Top10 = 0.7693 vs Top3 = 0.4952 → **targets DISAGREE** (both weak, Top3 is essentially random). The model can rank backmarker top-10 outcomes (engine failures, attrition) but cannot rank podium because podium positives are essentially absent.

### Summary

Targets **diverge most in extreme-imbalance regimes** (semi-street, front teams, backmarkers). In balanced regimes (2-stop, permanent circuits, midfield teams), they align well. This pattern is intentional and useful: the expansion target (`is_top3`) acts as a finer-grained ranking within top-10, and it degrades predictably when both positive and negative cases are skewed. The disagreement we care about for strategy recommendations is the **what-if scenario disagreement** (see `whatif_comparison.md`), where the PER Hungary 2023 1-stop M-S vs 2-stop M-H-H case shows the two targets recommending *opposite* strategies for the same driver-race context — the exact value the dual-target design is meant to surface.

---

## 5. Risks and Mitigations

| Risk | Severity | Evidence | Mitigation |
|------|----------|----------|-----------|
| Front-tier ROC-AUC near-random | **High** | ROC-AUC 0.6878 (top10), 0.7872 (top3) for Mercedes/RBR/Ferrari | Issue strategy advice for front teams *with caveats*: top10 is near-saturated, so strategy effects are dominated by inherent pace. Use is_top3 as the primary signal for front-tier strategy decisions. |
| Backmarker is_top3 uninformative | **High** | ROC-AUC 0.4952 (random); only 2 positive cases out of 312 | **Do not use** the is_top3 model for backmarker drivers. Use is_top10 instead (ROC-AUC 0.7693). |
| No-stop edge case | **Medium** | Only 15 test cases (1.7%); 0% top-10/top-3 finish rate | Flag no-stop scenarios as unreliable; require human review before deployment. |
| 3+-stop rarity | **Medium** | Only 153 test cases (17.2%); ROC-AUC 0.8638 for top10 (lower than 1/2-stop) | Flag 3+-stop scenarios with a confidence interval rather than a point estimate. |
| Semi-street top10 weakness | **Low** | ROC-AUC 0.8272 (top10) — lowest of the three circuit types, but still > 0.82 | Monitor performance; treat top10 numbers for semi-street with slightly wider error bars. is_top3 on semi-street is strong (ROC-AUC 0.9486). |

---

## 6. Conclusion

The model is **production-ready for most scenarios** but has clear blind spots:

1. **Front-tier constructor strategy advice** should rely on `is_top3` (ROC-AUC 0.7872) rather than `is_top10` (ROC-AUC 0.6878, near-random). Front teams finish top-10 too frequently (87.6%) for strategy choice to materially change the top-10 outcome.

2. **Backmarker `is_top3` is not deployable** (ROC-AUC 0.4952, essentially random). Use `is_top10` instead (ROC-AUC 0.7693) for backmarker tactical decisions.

3. **3+-stop strategies** should be flagged for limited training representation (ROC-AUC 0.8638 — usable but the weakest of 1/2/3+-stop on top10).

4. **No-stop scenarios** should be flagged as edge cases requiring human review (only 15 test cases, 0% top-10/top-3 finish rate).

5. **Preferred deployment zones** (ROC-AUC > 0.88 on at least one target): 2-stop strategies, permanent/street circuits, midfield teams. These slices have sufficient data, balanced class distributions, and reliable discrimination.

6. **Semi-street circuits** have slightly weaker top10 discrimination (ROC-AUC 0.8272) but very strong top3 discrimination (ROC-AUC 0.9486). Use confidently for `is_top3`; widen error bars for `is_top10`.

Future work could include **stratified model retraining** (separate models per circuit type or constructor tier) to recover performance in edge cases — out of scope for Hito 2 but noted in `mitigations.md`.
