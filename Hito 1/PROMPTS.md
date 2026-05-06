# IIT414W · Group 05 · HITO 1

This document records the development workflow used for HITO 1  in the format: Context → Prompt(s) → Output → Validation → Adaptations → Decision.
All design choices (feature selection, split design, model selection, regularisation strategy) were accepted only after empirical checks on the notebook outputs.
---


## Interaction 1: 

**1. Context:**  
We had a temporal split: train (2019‑2021), calibration (2022), test (2023‑2024). We wanted to calibrate the logistic regression probabilities using the 2022 season as a fixed calibration set.
Initial attempts used CalibratedClassifierCV(estimator=baseline_pipeline, method='isotonic', cv=5), but this performed 5‑fold cross‑validation inside the calibration set, which changed the mapping and made the calibration curve less stable

**2. Prompt(s):**  
> *"I have a separate calibration set (2022). I want to fit an isotonic calibration on exactly this set, not using cross‑validation. How can I do that with CalibratedClassifierCV? I tried cv='prefit' but got an error."*

**3. Relevant Output:**  
The assistant explained that in scikit‑learn ≥1.3, cv='prefit' is deprecated. For a single fixed calibration set, the correct approach is to set cv=None. This tells the calibrator to use the entire provided data for fitting the calibration curve without any splitting. It also warned that the estimator must already be fitted on the training set before passing it to CalibratedClassifierCV 

**4. Validation:**  
We set calibrated_model = CalibratedClassifierCV(estimator=baseline_pipeline, method='isotonic', cv=None). Then we called calibrated_model.fit(X_cal, y_cal). The calibration curve (Section 7) showed a nearly perfect diagonal after isotonic scaling, and the ECE dropped from 0.1769 to 0.0503. The model ran without errors and produced valid probabilities on the test set.

**5. Adaptations:**  
Used cv=None instead of cv=5 or cv='prefit'.

Ensured the baseline_pipeline was already fitted on the training data (2019‑2021) before calibration.

Kept the temporal separation intact; no leakage.

**6. Final Decision:**  
Used – cv=None is  correct and simplest way to calibrate on a dedicated validation set. The decision was kept in the final notebook.




## Interaction 2: How to compute Brier score per number of pit stops, and when to warn about small slices?

**1. Context:**  
We wanted to demonstrate how the model could support strategic decisions. In Section 10, we designed two scenarios (Monaco and Monza) to compare 1‑stop vs 2‑stop strategies. We created separate input rows for each strategy and used the calibrated model to predict P(Top10).

**2. Prompt(s):**  
> *"I have created two synthetic rows for Monaco: one with n_stops=1 and another with n_stops=2. I trained the model without n_stops because it was marked as leakage. Now the model gives the same probability for both rows. What’s wrong?"*

**3. Relevant Output:**  
The assistant pointed out that the model’s feature set (defined in Section 3) does not include n_stops or any strategy‑related variable. Therefore, changing the value of n_stops has no effect on predictions. Both input rows are identical on all features the model actually sees, so the output probabilities are forced to be equal. The assistant recommended either:

**4. Validation:**  
We manually inspected NUMERIC_FEATURES and confirmed that n_stops was absent. We printed the two scenario inputs side by side and saw they were identical. The predicted probabilities were exactly the same (0.7578 for Monaco, 1.0000 for Monza), confirming the issue.

**5. Adaptations:**  
We removed Section 10 from the final notebook because the comparison was invalid.

Added a note in the methodology section explaining that the current baseline model cannot evaluate pit‑stop count.

Planned for Hito 2 to either incorporate n_stops safely (e.g., using planned number of stops known before the race) or design a causal analysis.

---


### Interaction 3:

**1. Context:**  
We had a logistic regression model wrapped in a pipeline with a StandardScaler preprocessor. We wanted to explain individual predictions using SHAP waterfall plots, similar to the global SHAP bar plot we already generated. However, KernelExplainer requires a function that can take raw input and output probabilities. The pipeline’s predict method expects a DataFrame with original feature names, but the explainer passes NumPy arrays


**2. Prompt(s):**  
> *"I have a pipeline: preprocessor (StandardScaler) + LogisticRegression. I want to use SHAP KernelExplainer to explain predictions on my test set. The explainer expects a prediction function that accepts raw (unscaled) data. How can I wrap the pipeline so that it scales the input and then predicts probabilities? Also, I need to select specific test instances for waterfall plots."*

**3. Relevant Output:**  
The assistant provided a wrapper function:

python
def model_predict_proba(X):
    if not isinstance(X, pd.DataFrame):
        X = pd.DataFrame(X, columns=NUMERIC_FEATURES)
    X_transformed = baseline_pipeline.named_steps['preprocessor'].transform(X)
    return lr_model.predict_proba(X_transformed)[:, 1]
It also showed how to pick a single instance (e.g., idx_case1) and create a waterfall plot using shap.waterfall_plot(shap.Explanation(values=shap_values[idx_case1], ...))

**4. Validation:**  
We ran the wrapper and verified that model_predict_proba(X_test.iloc[[idx]]) returned the same probability as calibrated_model.predict_proba(X_test.iloc[[idx]]) (before calibration). The waterfall plots for the three chosen cases (correct top‑10, correct not top‑10, false positive) displayed correctly, with red/blue bars that summed to the difference from the base value

**5. Adaptations:**  

Implemented the wrapper function exactly as suggested.

Used shap.KernelExplainer with background of 100 random training samples.

Selected instances based on prediction confidence and actual labels to illustrate different scenarios.

Added feature names and title formatting for clarity.

**6. Final Decision:**  
Used – The SHAP waterfall plots are included in Section 9 and provide meaningful local explanations. The wrapper function is kept in the notebook






### Errors

What we tried:  
We built a what‑if scenario section (Section 10) to compare 1‑stop vs 2‑stop pit strategies for Monaco and Monza. We created separate inputs for each strategy and ran predictions.

What failed:
The model we trained does not include the `n_stops` feature (we excluded it because it could leak post‑race information). So both “1‑stop” and “2‑stop” inputs were identical as far as the model could see. The predictions came out exactly the same for both strategies (e.g., 0.7578 vs 0.7578).

Why it’s a problem:  
The section claimed to give strategy recommendations, but the model couldn’t actually tell the two scenarios apart. The “recommendation” was meaningless.

What we learned:  
Don’t simulate changes on variables that the model doesn’t use. If we want to compare pit strategies, we must either include `n_stops` safely (without leakage) or change the question to something the model can answer.

Next step:  
Remove Section 10 from the final notebook. Revisit this in Hito 2 with a proper causal approach.