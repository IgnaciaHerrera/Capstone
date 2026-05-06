# Team Decision Sheet — Capstone Hito 1
### IIT414W · F1 Race Strategy Advisor · Mon May 4, 2026

> **Instructions.** Complete this sheet in your team repo as `framing.md`. Every team has 60 minutes during the studio block (14:45–15:45). Required commits: by 15:00 (sections 1–4 populated) and by 15:40 (full sheet + dataset-load notebook). No section can be left blank — write "TBD with rationale" if you are uncertain, but blank entries fail the framing rubric.

**Team name:** Feligma
**Team members:** Martín Guerrero, Felipe Vázquez e Ignacia Herrera
**GitHub repo URL:** https://github.com/IgnaciaHerrera/Capstone.git

---

## 1. Decision Context

**What strategy decision is this tool supporting?**
> Whether to recommend a 1-stop, 2-stop, or 3-stop pit strategy for a given grid position and constructor to maximize the probability of finishing in the top 10, given expected tire degradation and pit stop time patterns observed historically.

**Who makes this decision?**
> The strategy engineer and strategy desk on the pit wall in coordination with the race engineer during pre-race strategy briefings and live strategy calls during the race weekend.

**When in the race weekend is the decision made?**
> Friday evening after FP2, when grid position is estimated and track conditions are known, allowing teams to set baseline strategy scenarios before Saturday qualifying and Sunday's race.

---

## 2. Target & Metric

**Target (LOCKED for Hito 1):** `is_top10`

**Primary metric:** Brier Score

**Why this metric for this decision?** (2 sentences max — what does the metric measure that an alternative does not?)

> Brier score measures calibration of predicted probabilities, not just ranking accuracy. For a scenario-comparison tool, we need to trust that P(top10)=0.75 actually means ~75% confidence in that outcome, so strategy desk can compare scenarios with calibrated uncertainty, not just point predictions.

**Secondary metric (optional but recommended):** ROC-AUC

**Temporal split (LOCKED for Hito 1):**
- Train: seasons 2019, 2020, 2021
- Calibration: season 2022 (used to fit calibration mapping; never for model selection)
- Test: seasons 2023, 2024 (untouched until final evaluation)

---

## 3. Baseline Plan

**Baseline approach (one sentence):**
> Calibrated logistic regression on 8 pre-race features (grid_position, qualifying_position, driver_prior3_avg_finish, constructor_prior3_avg_finish, driver_circuit_prior_avg, avg_track_temp, avg_air_temp, round) with Isotonic calibration applied on season 2022 calibration set.

**Why is this baseline F1-defendable?** (One sentence — could you justify it without ever seeing 2023–2024 data?)

> Grid and qualifying position reflect starting placement; prior 3-race driver/constructor averages capture recent momentum; circuit-specific driver history captures adaptation to different track types; temperature features encode track conditions affecting tire behavior—all observable before race-day strategy decisions are finalized.

**Direction check:** higher baseline score means higher predicted P(top10). Yes / No / Explain.

> Yes — predicted probability from logistic regression ranges [0,1] where 1 means model is maximally confident in top10 finish, and coefficient signs on grid_position and driver_prior3_avg_finish are expected to be negative (lower grid or worse history → lower P(top10)).

**Expected baseline performance vs docent floor:**
- Grid-rule docent baseline: Brier = 0.208 on test
- Calibrated docent model: Brier = 0.132 on test, ROC-AUC = 0.892
- Our team's expected baseline with 8-feature model: Brier ≤ 0.135, ROC-AUC ≥ 0.870

**LEAKAGE DECLARATION:**

> Strategy features such as `n_stops`, `compound_sequence`, `stint_lengths`, and `avg_pit_stop_duration_s` are post-race observations in the raw dataset and would constitute data leakage in a prediction task. However, in this capstone they serve as **user-controlled scenario inputs**, not as information known before the race. Our model receives these variables as intentional "what-if?" parameters set by the strategy desk *after* the race has concluded (in analysis mode) or projected *before* the race (in advisory mode). We do NOT use these as predictors during training; they are held out and only used at inference time to generate counterfactual comparisons. This design ensures that the baseline model in Section 3 is leakage-free.

---

## 4. What-If Comparison Plan

**Strategy variables we will vary:**
- [x] `n_stops`
- [x] `compound_sequence`
- [ ] `stint_lengths` (or stint1_length, stint2_length, etc.)
- [x] `avg_pit_stop_duration_s`
- [ ] Other:

**Concrete scenarios to compare (at least two, with specific values):**

> Scenario A: A midfield-grid driver (grid_position=8) at a high-downforce circuit (Monaco, circuit_type='street') with 1-stop M-H strategy, avg_pit_stop_duration=23.5s, vs. a 2-stop M-H-S strategy with avg_pit_stop_duration=24.2s. Which gives higher P(is_top10)?
> Scenario B: A front-grid driver (grid_position=2) at a high-speed circuit (Monza, circuit_type='high_speed') with 1-stop S-M strategy, avg_pit_stop_duration=22s, vs. a 2-stop M-M strategy with avg_pit_stop_duration=23.5s. Which gives higher P(is_top10)?

**Decision metric for the comparison:**
> Difference in calibrated P(is_top10) between scenario A (1-stop) and scenario A (2-stop), reported with 90% bootstrap confidence interval; whichever scenario has higher point estimate and lower confidence interval lower bound is recommended as higher-confidence choice. Model must support filtering by circuit_type to enable scenario-specific comparisons (e.g., predict separately for street vs. high-speed circuits).

---

## 5. Limitations Acknowledgment

**Five known dataset limitations are documented in the Capstone Brief. Which TWO most affect our team's specific approach?**

Limitation #1 we acknowledge: Qualifying position is a stand-in for true grid position; qualifying_time_s is empty.

> Why it matters for our approach (1 sentence): Grid position is our #1 predictor, but using qualifying_position as a proxy rather than race-day grid-determined-by-penalty introduces measurement error that could attenuate the coefficient and reduce model discriminant power between tight grid clusters.

Limitation #2 we acknowledge: Strategy choice is confounded with car pace, driver skill, weather, and race incidents.

> Why it matters for our approach (1 sentence): We cannot infer causality from n_stops → is_top10 because teams that are already fast may choose 1-stop strategies out of confidence, making it impossible to isolate whether the 1-stop *caused* the top10 or the fast car/driver *chose* the 1-stop; our what-if scenarios assume historical correlations hold but cannot prove counterfactual causation.

**Mitigation Plan for Milestone 2:** To surface whether strategy choice bias varies by circuit context, we will perform error analysis segmented by `circuit_type` (street vs. high-speed vs. hybrid), comparing model residuals for 1-stop vs. 2-stop drivers within each circuit type. If 1-stop residuals are systematically more positive on high-speed circuits and negative on street circuits, this suggests the model is learning true circuit-strategy interactions rather than pure confounding; if residuals are uniform, we will flag the confounding as severe and caveat recommendations appropriately.

---

## 6. Experiment Plan for Hito 1

1. **Temporal validation and Brier score** on train/cal/test split: Retrain baseline logistic regression on 2019–2021, calibrate on 2022 with Isotonic scaling, evaluate on 2023–2024. Verify that model inputs (grid_position, qualifying_position, driver_prior3_avg_finish, constructor_prior3_avg_finish, driver_circuit_prior_avg, avg_track_temp, avg_air_temp, round) are available for all rows.
2. **Calibration curve and ECE (Expected Calibration Error)**: Plot reliability diagram on season 2023 hold-out to verify post-calibration miscalibration is reduced vs. raw logistic output. Isotonic calibration will be fitted on season 2022 and frozen before test-set evaluation.
3. **What-if scenario comparison on Monza 2023 high-speed circuit**: Manually set n_stops and compound_sequence for actual Monza starters, filter by circuit_type='high_speed', and compare predicted P(is_top10) across 1-stop vs. 2-stop scenarios using the fitted baseline + Isotonic calibration

**Hypothesis for each (one line each — what do we expect to happen and why?):**

> 1. Brier score will drop from ~0.205 (raw logistic) to ~0.131 (after Isotonic calibration) on 2023–2024 test set because the 8-feature model captures complementary pre-race signals: grid/qualifying position (start placement), recent driver/constructor form (momentum), circuit-specific history (adaptation), and temperature (track conditions).
> 2. Post-calibration ECE (Expected Calibration Error) will fall below 0.075 because Isotonic calibration fits a piecewise-linear mapping to observed frequencies in the season 2022 calibration set, removing systematic over/under-confidence more flexibly than Platt's sigmoid.
> 3. For Monza 2023 (high-speed circuit), 1-stop strategies will show higher predicted P(is_top10) than 2-stop on average (ΔP ≈ +0.08 to +0.12) because shorter pit-time exposure and maintained tire grip in later stints favor fewer stops on fast circuits; however, confidence intervals will overlap by ~±0.06 due to confounding with driver skill and car setup (which historically co-vary with stop strategy choice).

**Fallback Plan (If model does not beat baseline):**

> If Brier ≥ 0.20 (fails to beat docent grid-rule baseline of 0.208): We will analyze residuals segmented by `circuit_type` and `strategy_type` (n_stops) to identify which circuit/strategy combinations the model misses most, and hypothesize whether the limiting factor is confounding (strategy choice biased by fast cars), insufficient feature engineering (missing tire compound predictiveness), or data coverage (insufficient 2023-2024 data for rare strategy patterns). This analysis will surface whether the model has learned meaningful strategy interactions or is simply replicating team pace, and will inform recommendations for Hito 2 (e.g., add team-fixed effects, boost sample size with international series data, or switch to ranking-based targets if calibration proves intractable).


---

## 7. Team Workflow

**Who is doing what between now and Wednesday?**

| Member | Owns | Branch / file in repo |
|---|---|---|
| Ignacia | Data load + feature verification | hito1_baseline.ipynb |
| Felipe | Baseline model + pipeline  | hito1_baseline.ipynb |
| Martín | Calibration + what-if scenarios | hito1_baseline.ipynb |
| Group 5 | Integration + cross-temporal validation | hito1_baseline.ipynb |

**When does each member commit by?** (We need at least one commit per member per day Tue and Wed.)

| Member | Wednesday Commit |
|---|---|
| Ignacia | 14:00 (data load complete + feature verification) |
| Felipe | 14:30 (baseline model + pipeline tested) |
| Martín | 15:00 (calibration + what-if scenarios executable) |
| Group (Integration) | 15:30 (integration + cross-temporal validation complete) |

---

## 8. Critique Received in Pair Review

> *Filled during Block 5 (15:45–16:05) after the partner team reviews this sheet.*

**Reviewing team:** Group 20

**Concrete critique we received:**

> Your Section 3 Leakage Declaration states you will not use strategy features like n_stops during training but will use them at inference for your what-if scenarios. The consequence is that your model mathematically cannot generate different predictions for your scenarios because it won't have learned weights for features it wasn't trained on, completely breaking your Hito 1 experiment. One thing to do: include the strategy variables in the training set and properly acknowledge the post-race leakage limitation in Section 5 instead of trying to avoid it.

**How we will address this critique by Wednesday:**

> We evaluated the critique and chose a different resolution: rather than adding strategy variables to training (which would require acknowledging post-race leakage as a core model limitation), we restructured the baseline to use 8 purely pre-race features (grid_position, qualifying_position, driver/constructor prior averages, circuit history, temperatures, round). What-if scenario comparisons will be executed as post-hoc lookups over the historical dataset — filtering rows by n_stops and circuit_type and comparing their observed calibrated P(is_top10) distributions — rather than as model inputs. This keeps the baseline leakage-free and the scenarios interpretable as historical correlation analysis, not causal inference.

---

## Self-Check Before Committing

Before you push this to GitHub, verify:

- [ ] Decision context is one sentence, not a paragraph
- [ ] Target says exactly `is_top10` (not "Top-10" or "P(top10)")
- [ ] Temporal split shows three blocks: 2019–2021 / 2022 / 2023–2024
- [ ] Baseline is described in code-realistic terms (we could implement it)
- [ ] What-if scenarios have specific feature values, not generic words
- [ ] At least 2 of the 5 limitations are acknowledged with consequence
- [ ] PROMPTS.md exists in the repo (even if empty for now — will be populated by Wednesday)
