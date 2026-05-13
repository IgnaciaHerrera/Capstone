# AI Assistance Documentation � Hito 2 & Hito 1

This document records the development workflow used for HITO 2 and HITO 1. For each component, it records: Context ? Prompt(s) ? Output ? Validation ? Adaptations ? Decision.

## HITO 2: Dual-Target Model Updates

### 1. Implement the Expansion Target's Pipeline (\is_top3\)
**Context:** Required to add a second target (\is_top3\) using the same feature set (\CORRECTED_FEATURES\) and temporal split, ensuring a fair side-by-side comparison with the primary target (\is_top10\).
**Prompt:** "I need to train a second logistic regression model for 'is_top3'. Provide the pipeline implementation using the existing preprocessor, fit it on the 2019-2021 train set, and calibrate it on the 2022 dataset just like the primary target."
**AI Output:** Suggested duplicating the entire preprocessing block and re-fitting the scaler/imputer specifically for \is_top3\ before training the classifier.
**Validation:** Evaluated on \is_top3\. The model ran, but maintaining two separate preprocessors caused code bloat and risk of feature misalignment.
**Adaptations (Corrections made):** We *rejected* the AI's redundant preprocessor. Instead, we shared the exact same \uild_preprocessor_s0()\ pipeline state, branching only at the classifier level (\LogisticRegression(class_weight='balanced')\). 
**Decision:** Shared a single preprocessor to guarantee both models evaluate the exact same transformed feature distributions.

### 2. Generate Error-Analysis Slicing Code
**Context:** Need to slice test-set errors (Brier, ROC-AUC) by multiple dimensions (n_stops, circuit_type, constructor_tier).
**Prompt:** "Write a pandas grouping function to slice our test DataFrame by 'n_stops', 'circuit_type', and 'constructor_tier'. For each slice, compute Brier score and ROC-AUC for both 'is_top10' and 'is_top3'."
**AI Output:** Provided a function to group by values, but didn't handle ROC-AUC errors when a slice has only 1 class.
**Validation:** Code crashed with \ValueError\ when 0-stop strategies had zero positive cases limits.
**Adaptations:** Added a try-except block and a length check (\if len(np.unique(y_true)) > 1\) to return \
p.nan\ instead of crashing.
**Decision:** Kept the fixed, robust slicing function.

### 3. Validate Calibration / Probability Quality
**Context:** Need to evaluate how well-calibrated the \is_top3\ probabilities are compared to \is_top10\.
**Prompt:** "Generate an evaluation block that computes the Brier score, ROC-AUC, and plots a calibration curve for both 'is_top10' and 'is_top3' on the test set."
**AI Output:** Provided code using default \
_bins=10\ for the calibration curve.
**Validation:** The calibration curve was extremely erratic for \is_top3\ at probabilities > 0.4 because the base rate is only ~15% for podiums.
**Adaptations:** Reduced \
_bins\ and explicitly computed the Brier score against a "grid-only baseline", since guessing near 0% naturally yields an artificially low Brier.
**Decision:** Adjusted visualization logic for sparsity and included the baseline comparison.

### 4. Reason About Strategy Confounding
**Context:** The strategy features (\
_stops\) correlate heavily with inherent car pace (e.g., front-tier cars stop less).
**Prompt:** "Write a markdown section explaining the confounding limitation between strategy choice and car pace for F1. How does this affect our what-if scenario predictions?"
**AI Output:** Advised *dropping* \
_stops\ from the model entirely to fix the bias.
**Validation:** Dropping the strategy inputs would violate the core requirement of building a "What-If" strategy advisor for Hito 2.
**Adaptations:** We *rejected* the AI's advice to drop the columns. Instead, we adapted the error analysis slicing by \constructor_tier\ to demonstrate the confounding quantitatively: backmarker cars showed a near-random ROC-AUC for \is_top3\.
**Decision:** Strategy features were kept, but we documented a hard deployment mitigation: "Do not deploy \is_top3\ predictions for backmarker teams."

---

# IIT414W � Group 05 � HITO 1 (Legacy Prompts)

This document records the development workflow used for HITO 1 in the format: Context ? Prompt(s) ? Output ? Validation ? Adaptations ? Decision.
All design choices (feature selection, split design, model selection, regularisation strategy) were accepted only after empirical checks on the notebook outputs.

---

## Interaction 1: 

**1. Context:**  
We had a temporal split: train (2019-2021), calibration (2022), test (2023-2024). We wanted to calibrate the logistic regression probabilities using the 2022 season as a fixed calibration set.
Initial attempts used CalibratedClassifierCV(estimator=baseline_pipeline, method='isotonic', cv=5), but this performed 5-fold cross-validation inside the calibration set, which changed the mapping and made the calibration curve less stable

**2. Prompt(s):**  
> *"I have a separate calibration set (2022). I want to fit an isotonic calibration on exactly this set, not using cross-validation. How can I do that with CalibratedClassifierCV? I tried cv='prefit' but got an error."*

**3. Relevant Output:**  
The assistant explained that in scikit-learn >=1.3, cv='prefit' is deprecated. For a single fixed calibration set, the correct approach is to set cv=None. This tells the calibrator to use the entire provided data for fitting the calibration curve without any splitting. It also warned that the estimator must already be fitted on the training set before passing it to CalibratedClassifierCV 

**4. Validation:**  
We set calibrated_model = CalibratedClassifierCV(estimator=baseline_pipeline, method='isotonic', cv=None). Then we called calibrated_model.fit(X_cal, y_cal). The calibration curve (Section 7) showed a nearly perfect diagonal after isotonic scaling, and the ECE dropped from 0.1769 to 0.0503. The model ran without errors and produced valid probabilities on the test set.

**5. Adaptations:**  
Used cv=None instead of cv=5 or cv='prefit'.
Ensured the baseline_pipeline was already fitted on the training data (2019-2021) before calibration.
Kept the temporal separation intact; no leakage.

**6. Final Decision:**  
Used - cv=None is correct and simplest way to calibrate on a dedicated validation set. The decision was kept in the final notebook.

## Interaction 2: How to compute Brier score per number of pit stops, and when to warn about small slices?

**1. Context:**  
We wanted to demonstrate how the model could support strategic decisions. In Section 10, we designed two scenarios (Monaco and Monza) to compare 1-stop vs 2-stop strategies. We created separate input rows for each strategy and used the calibrated model to predict P(Top10).

**2. Prompt(s):**  
> *"I have created two synthetic rows for Monaco: one with n_stops=1 and another with n_stops=2. I trained the model without n_stops because it was marked as leakage. Now the model gives the same probability for both rows. What's wrong?"*

**3. Relevant Output:**  
The assistant pointed out that the model's feature set (defined in Section 3) does not include n_stops or any strategy-related variable. Therefore, changing the value of n_stops has no effect on predictions. Both input rows are identical on all features the model actually sees, so the output probabilities are forced to be equal. The assistant recommended either:

**4. Validation:**  
We manually inspected NUMERIC_FEATURES and confirmed that n_stops was absent. We printed the two scenario inputs side by side and saw they were identical. The predicted probabilities were exactly the same (0.7578 for Monaco, 1.0000 for Monza), confirming the issue.

**5. Adaptations:**  
We removed Section 10 from the final notebook because the comparison was invalid.
Added a note in the methodology section explaining that the current baseline model cannot evaluate pit-stop count.
Planned for Hito 2 to either incorporate n_stops safely (e.g., using planned number of stops known before the race) or design a causal analysis.

---

### Interaction 3:

**1. Context:**  
We had a logistic regression model wrapped in a pipeline with a StandardScaler preprocessor. We wanted to explain individual predictions using SHAP waterfall plots, similar to the global SHAP bar plot we already generated. However, KernelExplainer requires a function that can take raw input and output probabilities. The pipeline's predict method expects a DataFrame with original feature names, but the explainer passes NumPy arrays

**2. Prompt(s):**  
> *"I have a pipeline: preprocessor (StandardScaler) + LogisticRegression. I want to use SHAP KernelExplainer to explain predictions on my test set. The explainer expects a prediction function that accepts raw (unscaled) data. How can I wrap the pipeline so that it scales the input and then predicts probabilities? Also, I need to select specific test instances for waterfall plots."*

**3. Relevant Output:**  
The assistant provided a wrapper function:

\\\python
def model_predict_proba(X):
    if not isinstance(X, pd.DataFrame):
        X = pd.DataFrame(X, columns=NUMERIC_FEATURES)
    X_transformed = baseline_pipeline.named_steps['preprocessor'].transform(X)
    return lr_model.predict_proba(X_transformed)[:, 1]
\\\
It also showed how to pick a single instance (e.g., idx_case1) and create a waterfall plot using shap.waterfall_plot(shap.Explanation(values=shap_values[idx_case1], ...))

**4. Validation:**  
We ran the wrapper and verified that model_predict_proba(X_test.iloc[[idx]]) returned the same probability as calibrated_model.predict_proba(X_test.iloc[[idx]]) (before calibration). The waterfall plots for the three chosen cases (correct top-10, correct not top-10, false positive) displayed correctly, with red/blue bars that summed to the difference from the base value

**5. Adaptations:**  
Implemented the wrapper function exactly as suggested.
Used shap.KernelExplainer with background of 100 random training samples.
Selected instances based on prediction confidence and actual labels to illustrate different scenarios.
Added feature names and title formatting for clarity.

**6. Final Decision:**  
Used - The SHAP waterfall plots are included in Section 9 and provide meaningful local explanations. The wrapper function is kept in the notebook

## Interaction 5: 
**Context:**
We were building a baseline logistic regression model to predict top-10 finishes. The dataset contained 47 columns, many of which were only known after the race (e.g., number of pit stops, stint lengths, finishing position). To avoid data leakage, we needed to separate pre-race (safe) from post-race (leakage) features. We also wanted to start with a small set of numeric features that would be easy to interpret and standardize.

**Prompt(s):**
> "We have a dataset with 47 columns: season, round, race_name, circuit_id, driver_id, grid_position, qualifying_position, n_stops, stint_lengths, finish_position, points, is_top10, dnf, safety_car_periods, wet_laps, etc. Help me categorise them into pre-race (available before the race) and post-race (only known after). Also, suggest 8 numeric pre-race features for a simple logistic regressi..."
