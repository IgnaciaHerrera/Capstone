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
> Calibrated logistic regression on grid_position + constructor_tier + driver_prior3_avg_finish + circuit_type, with Platt scaling applied on season 2022 calibration set.

**Why is this baseline F1-defendable?** (One sentence — could you justify it without ever seeing 2023–2024 data?)

> In F1, grid position is the #1 predictor of race outcome; constructor tier determines car potential; recent driver form predicts consistency; and circuit type (street vs high-speed) affects which teams/drivers excel, all known before race day strategy is decided.

**Direction check:** higher baseline score means higher predicted P(top10). Yes / No / Explain.

> Yes — predicted probability from logistic regression ranges [0,1] where 1 means model is maximally confident in top10 finish, and coefficient signs on grid_position and driver_prior3_avg_finish are expected to be negative (lower grid or worse history → lower P(top10)).

**Expected baseline performance vs docent floor:**
- Grid-rule docent baseline: Brier = 0.208 on test
- Calibrated docent model: Brier = 0.132 on test, ROC-AUC = 0.892
- Our team's best baseline expected to land near or lower: Brier = 0.129

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

**T**Temporal validation and Brier score** on train/cal/test split: Retrain baseline logistic regression on 2019–2021, calibrate on 2022 with Platt scaling, evaluate on 2023–2024. Verify that model inputs (grid_position, constructor_tier, driver_prior3_avg_finish, circuit_type) are available for all rows.
2. **Calibration curve and ECE (Expected Calibration Error)**: Plot reliability diagram on season 2023 hold-out to verify post-calibration miscalibration is reduced vs. raw logistic output. Ensure circuit_type is encoded (e.g., one-hot or numeric) so it's usable in the scenario framework.
3. **What-if scenario comparison on Monza 2023 high-speed circuit**: Manually set n_stops and compound_sequence for actual Monza starters, filter by circuit_type='high_speed', and compare predicted P(is_top10) across 1-stop vs. 2-stop scenarios using the fitted baseline + Platt calibration

**Hypothesis for each (one line each — what do we expect to happen and why?):**

> 1. Brier score will drop from ~0.200 (raw) to ~0.145 (after Platt calibration) on 2023–2024 test set because grid_position and constructor_tier are stable pre-race signals with low variance across seasons, and circuit_type partitions races into learnable subgroups.
> 2. Post-calibration ECE (Expected Calibration Error) will fall below 0.08 because Platt scaling fits a sigmoid directly to calibration set probabilities (season 2022), removing systematic over/under-confidence observed in raw logistic outputs.
> 3. For Monza 2023 (high-speed circuit), 1-stop strategies will show higher predicted P(is_top10) than 2-stop on average (ΔP ≈ +0.08 to +0.12) because shorter pit-time exposure and maintained tire grip in later stints favor fewer stops on fast circuits; however, confidence intervals will overlap by ~±0.06 due to confounding with driver skill and car setup (which historically co-vary with stop strategy choice).


---

## 7. Team Workflow

**Who is doing what between now and Wednesday?**

| Member | Owns | Branch / file in repo |
|---|---|---|
|  |  |  |
|  |  |  |
|  |  |  |

**When does each member commit by?** (We need at least one commit per member per day Tue and Wed.)

> 

---

## 8. Critique Received in Pair Review

> *Filled during Block 5 (15:45–16:05) after the partner team reviews this sheet.*

**Reviewing team:** ____________________

**Concrete critique we received:**

> 

**How we will address this critique by Wednesday:**

> 

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
