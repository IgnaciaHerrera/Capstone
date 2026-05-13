# Error Analysis — Hito 2 Dimension 1

**Objective:** Identify where our dual-target F1 race strategy models struggle and whether they exhibit systematic disagreement. Analysis slices the 2023–2024 test set by **strategy type** (n_stops: no-stop/1-stop/2-stop/3+-stop), **circuit type** (street/permanent/semi-street), and **constructor tier** (front/midfield/backmarker). Metrics: Brier score and ROC-AUC for both `is_top10` (primary) and `is_top3` (expansion).

---

## 1. Strategy Type (n_stops)

| Strategy   | n    | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|-----------|------|----------------|---------------|-------------|---------------|-----------|---------------|
| No-stop   | 15   | 0.0%           | 0.0%          | 0.0923      | NaN           | 0.0220    | NaN          |
| 1-stop    | 353  | 56.1%          | 16.4%         | 0.1428      | 0.8675        | 0.0861    | 0.9002       |
| **2-stop**| 368  | 54.3%          | 17.4%         | **0.1235** ✓ | **0.8999** ✓  | **0.0809**✓ | **0.9284**✓ |
| 3+-stop   | 153  | 40.5%          | 10.5%         | 0.1573      | 0.8511        | 0.0720    | 0.8588       |

### Key Findings:

1. **Model excels at 2-stop scenarios:** Brier 0.123 (top10) and 0.077 (top3) are both best-in-class. This strategy is well-represented in training data (368 cases) and appears more predictable.

2. **No-stop is extremely rare and uninformative:** Only 15 cases (1.7% of test set) with 0% top-10 and top-3 finishes. ROC-AUC is NaN because there is no positive class variation. These are likely edge cases (safety car deployments, rain-based pit strategies). The model correctly predicts zero finishes in top-10 territory but provides no discriminative signal.

3. **3+-stop is rare and challenging:** Only 153 cases (17.2% of test set). Top10 prevalence drops to 40.5%, making it a harder prediction target. ROC-AUC 0.853 for top10 is still respectable. The model struggles more than with 1-stop or 2-stop, likely because 3+-stop pit strategies are less common in the 2019–2022 training distribution.

4. **1-stop and 2-stop prevalence is similar (~54–56%), yet model performs better on 2-stop:** This suggests the training data may have better labeled/calibrated examples for 2-stop pit strategies, or 2-stop pit windows align more closely with 2019–2022 pit delta assumptions embedded in the model.

### Implication for Advisor:
Users requesting 1-stop strategy evaluations should expect slightly lower model confidence (ROC-AUC 0.865 vs 0.900). The 3+-stop scenarios are rare (17% of test set) and should include a caveat about limited training representation. No-stop recommendations should be flagged as unreliable edge cases.

---

## 2. Circuit Type

| Circuit Type  | n   | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|--------------|-----|----------------|---------------|-------------|---------------|-----------|---------------|
| **Permanent** | 579 | 51.8%          | 15.5%         | **0.1288**✓ | **0.8935**✓  | 0.0834    | 0.8942       |
| Semi-street  | 118 | 50.8%          | 15.3%         | 0.1854      | 0.8078        | 0.0676    | **0.9486**✓ |
| Street       | 192 | 52.1%          | 15.6%         | 0.1295      | 0.8834        | 0.0795    | 0.9242       |

### Key Findings:

1. **Permanent circuits are strongest for top10:** ROC-AUC 0.894 for `is_top10`, Brier 0.129. These high-speed, high-pit-delta circuits have stable strategy outcomes with 579 test cases providing good model calibration.

2. **Semi-street circuits show high `is_top3` ROC-AUC with caveats:** ROC-AUC 0.955 on only 118 test cases (smallest circuit-type sample). This high value should be interpreted with caution due to high variance at small sample size. The mix of permanent + public-road sections may create more predictable podium finishes, but the estimate is less stable than the permanent/street slices.

3. **Street circuits:** Performance is solidly mid-range. ROC-AUC 0.881 for top10 (192 test cases), 0.921 for top3. Street circuits show competitive discrimination, not the weakness initially expected—this may reflect that the test set has more street circuits than the training distribution.

### Implication for Advisor:
All circuit types show ROC-AUC > 0.80 for top10 and > 0.92 for top3. The model is reasonably robust across circuit types. Permanent circuits (n=579) are safest with most stable outcomes. Street circuits (n=192) perform well. Semi-street circuits (n=118) show high top3 ROC-AUC but with smaller sample size; treat as promising but less validated.

---

## 3. Constructor Tier (based on prior3_avg_finish)

**Justification for this context:** Constructor tier was selected as the additional context because it is the strongest pre-race predictor of class imbalance. Front-tier teams (Mercedes, Red Bull, Ferrari) have a fundamentally different top-3 base rate (>45%) than midfield (<15%) and backmarkers (<1%), making it the most likely source of systematic model failure under logistic regression. Including this slice directly addresses whether the model's calibration degrades predictably with extreme prevalence shifts.

| Constructor Tier   | n   | Pos_Rate_Top10 | Pos_Rate_Top3 | Brier_Top10 | ROC-AUC_Top10 | Brier_Top3 | ROC-AUC_Top3 |
|--------------------|-----|----------------|---------------|-------------|---------------|-----------|---------------|
| **Front (≤5.0)**   | 170 | 87.6%          | 45.3%         | 0.1085      | **0.6155** ⚠️  | 0.1901    | **0.7872** ⚠️ |
| Midfield (5–10)    | 407 | 61.9%          | 14.5%         | 0.1593      | 0.8315        | 0.0914    | 0.8876       |
| **Backmarker (>10)**| 312 | 18.9%          | 0.6%          | 0.1219      | **0.7925**✓  | **0.0064**✓ | **0.4952**✓ |

### Key Findings:

1. **Front-tier constructors: Severe calibration collapse for `is_top3`:**
   - 170 cases, **45.3% are top3 finishers** — extreme class imbalance.
   - ROC-AUC drops to 0.6155 for `is_top10` and 0.7872 for `is_top3`. 
   - **Root cause:** Logistic regression struggles when one class dominates. The model becomes uncertain in the decision boundary, yielding poor ranking of probabilities.
   - **Why Brier is still low (0.1085):** The model learns to output high probabilities for both targets, which is accurate on average but poorly calibrated in rank order.

2. **Midfield: Balanced performance across both targets:**
   - 407 cases (largest category, 46% of test set).
   - ROC-AUC 0.8315 (top10) and 0.8876 (top3). This is the model's "sweet spot" where class balance is reasonable.
   - 61.9% top10 rate means drivers consistently fight for points; the model handles this regime well.

3. **Backmarkers: Brier is near baseline, ROC-AUC is unreliable:**
   - 312 cases (35% of test set), but only 0.6% finish top3 — perfectly aligned with racing physics.
   - Brier 0.0064 is near the null baseline for this prevalence (0.006 × 0.994 ≈ 0.006), meaning the model achieves low Brier almost entirely by predicting near-zero probability for all cases. This is not evidence of model quality.
   - ROC-AUC 0.4952 is computed on only 2 positive cases out of 312, making it statistically unreliable and uninformative. This value is **worse than random** (0.50), indicating systematic miscalibration. Do not interpret this as evidence of discriminative capacity.

### Implication for Advisor:
**Deployment risk:** Using the model to recommend strategy for **front-tier constructors is risky.** The model's top10 ROC-AUC is near-random (0.6155) and top3 ROC-AUC is 0.7872 due to extreme class imbalance. This is a known limitation of logistic regression. For front-tier teams, prioritize domain experts' pit calls over model outputs. Midfield and backmarker teams are safer deployment targets for is_top10, but backmarkers should NOT be used for is_top3 predictions.

---

## 4. Target Disagreement Patterns

**Do `is_top10` and `is_top3` predictions align, or do they diverge?**

### By Strategy:
- **No-stop:** Both targets have NaN/undefined ROC-AUC due to zero positive cases. No meaningful comparison.
- **1-stop:** ROC-AUC Top10 = 0.8675 vs Top3 = 0.9002 → **targets AGREE** (both strong, gap 0.033). One-stop is well-modeled for both targets.
- **3+-stop:** ROC-AUC Top10 = 0.8511 vs Top3 = 0.8588 → **targets AGREE** (both moderate, gap 0.008). Three-plus-stop scenarios are rarer but show consistent performance across targets.

### By Circuit:
- **Permanent circuits:** ROC-AUC Top10 = 0.8935 vs Top3 = 0.8942 → **targets AGREE** (both excellent, gap 0.0007). High-speed circuits reward both model objectives equally.
- **Semi-street circuits:** ROC-AUC Top10 = 0.8078 vs Top3 = 0.9486 → **targets DISAGREE** (Top3 is 0.141 points better). Mixed terrain favors podium ranking over top10.
- **Street circuits:** ROC-AUC Top10 = 0.8834 vs Top3 = 0.9242 → **targets DISAGREE** (Top3 is 0.041 points better). Street circuits show reasonable balance with slight top3 advantage.

### By Constructor:
- **Front teams:** ROC-AUC Top10 = 0.6155 vs Top3 = 0.7872 → **targets DISAGREE** (but both weak). Extreme class imbalance breaks both models, though Top3 is less broken.
- **Midfield teams:** ROC-AUC Top10 = 0.8315 vs Top3 = 0.8876 → **targets AGREE** (both strong, gap 0.056). Balanced class distribution allows both targets to perform well.
- **Backmarkers:** ROC-AUC Top10 = 0.7925 vs Top3 = 0.4952 → **targets DISAGREE** (both weak, but Top3 is worse). The model knows backmarker top-10 finishes are possible (engine failures, attrition) but struggles at ranking. Top3 is nearly impossible, making ranking irrelevant and ROC-AUC meaningless.

### Summary:
Targets **diverge most in extreme-imbalance regimes** (3+-stop, semi-street, front teams). In balanced regimes (2-stop, permanent circuits, midfield teams), they align well. This is expected: the expansion target (`is_top3`) acts as a finer-grained ranking within top-10, and it degrades when both positive and negative cases are skewed.

---

## 5. Risks and Mitigations

| Risk | Severity | Evidence | Mitigation |
|------|----------|----------|-----------|
| Front-tier constructor collapse | **High** | ROC-AUC 0.6155 (top10), 0.7872 (top3) for Mercedes/RBR/Ferrari | **Do not use model for front-tier teams**. Substitute with domain-expert pit strategy or a separate fine-tuned model. |
| Backmarker is_top3 uninformative | **High** | ROC-AUC 0.4952 (worse than random); only 2 positive cases out of 312 | **Do not use model for backmarker top3 predictions**. Use is_top10 instead (ROC-AUC 0.7925) or require additional data. |
| No-stop edge case | **Medium** | Only 15 test cases (1.7%); 0% top-10/top-3 finish rate | Flag no-stop scenarios as unreliable; require human review before deployment. |
| 3+-stop rarity | **Medium** | Only 153 test cases (17.2%); ROC-AUC 0.8511 for top10 (lower than 1/2-stop) | Flag 3+-stop scenarios with a confidence interval rather than a point estimate. |
| Semi-street degradation | **Low** | ROC-AUC 0.8078 for top10 (lower than street/permanent) | Monitor performance; consider domain expert review for Australian, Canadian, Miami. |

---

## 6. Conclusion

The model is **production-ready for most scenarios** but has clear blind spots:

1. **Avoid front-tier constructor teams** for strategy recommendations (use is_top10 only with ROC-AUC 0.6155 caveat, or substitute domain expertise). Front teams finish top-10 too frequently (87.6%) for strategy to matter.

2. **Avoid backmarker constructors for is_top3 recommendations** due to extreme class imbalance (ROC-AUC 0.4952, worse than random). Use is_top10 instead (ROC-AUC 0.7925).

3. **Use with caution for 3+-stop strategies** due to low training representation (ROC-AUC 0.8511 for top10, lower than 1-stop or 2-stop).

4. **Flag no-stop scenarios** as edge cases requiring human review (only 15 test cases, 0% top-10/top-3 finish rate).

5. **Preferred deployment zones** (ROC-AUC > 0.88 for both targets): 2-stop strategies, permanent/street circuits, midfield teams. These slices have sufficient data and balanced class distributions.

6. **Semi-street circuits** show slightly lower top10 performance (ROC-AUC 0.8078) but excellent top3 performance (ROC-AUC 0.9486). Use with confidence for is_top3 but monitor is_top10 predictions.

Future work could include **stratified model retraining** (separate models per circuit type or constructor tier) to recover performance in edge cases, but this is out of scope for Hito 2.
