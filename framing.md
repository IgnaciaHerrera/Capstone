# \Team Decision Sheet — Capstone Hito 1

### IIT414W · F1 Race Strategy Advisor · Mon May 4, 2026

> **Instructions.** Complete this sheet in your team repo as `framing.md`. Every team has 60 minutes during the studio block (14:45–15:45). Required commits: by 15:00 (sections 1–4 populated) and by 15:40 (full sheet + dataset-load notebook). No section can be left blank — write "TBD with rationale" if you are uncertain, but blank entries fail the framing rubric.

**Team name:** Group 20
**Team members:** Martin Yungue & Benjamin Bennett
**GitHub repo URL:** https://github.com/MYR2003/iit414w-lab01-Group20/

---

## 1. Decision Context

**What strategy decision is this tool supporting?**

> Whether a driver will realistically score championship points (finish top-10) in the upcoming race, determining whether the team should pursue an aggressive points-seeking strategy or a conservative race-survival/setup-testing strategy.

**Who makes this decision?**

> The Team Principal and Strategy Director on Friday evening, in conjunction with the driver to finalize pit-stop tactics and tire compound allocation before Friday night strategy meetings.

**When in the race weekend is the decision made?**

> Friday evening after Free Practice 2, once grid-position rumors are known and team performance baseline is established, but before final car setup freeze and parc fermé conditions (Saturday evening).

---

## 2. Target & Metric

**Target (LOCKED for Hito 1):** `is_top10` (binary: 1 if driver finishes in positions 1–10 earning points, 0 if positions 11+ or DNF)

**Primary metric:** Brier Score (mean squared error between predicted probability and actual outcome)

**Why this metric for this decision?** (2 sentences max — what does the metric measure that an alternative does not?)

> Brier Score measures calibration—whether a model's predicted P(top-10) actually matches real-world frequencies—critical for a strategist to trust the model's confidence. Unlike accuracy, Brier Score penalizes incorrect confidence (e.g., predicting 90% when outcome is 0%), making it honest about when the model is uncertain and when it is reliable.

**Secondary metric (optional but recommended):** ROC-AUC (area under receiver-operating-characteristic curve) to measure rank-ordering of drivers by true top-10 likelihood

**Temporal split (LOCKED for Hito 1):**

- Train: seasons 2019, 2020, 2021
- Calibration: season 2022 (used to fit calibration mapping; never for model selection)
- Test: seasons 2023, 2024 (untouched until final evaluation)

---

## 3. Baseline Plan

**Baseline approach (one sentence):**

> Logistic regression using grid position and constructor tier as features to predict P(is_top10), with calibration mapping applied post-prediction to align probabilities with observed frequencies.

**Why is this baseline F1-defendable?** (One sentence — could you justify it without ever seeing 2023–2024 data?)

> Grid position and constructor strength are the two most transparent pre-race predictors of race outcome; a logistic regression on these features requires no hyperparameter tuning and reflects the sport's fundamental performance hierarchy.

**Direction check:** higher baseline score means higher predicted P(top10). Yes / No / Explain.

> Yes. Higher grid position (P1 > P20) and stronger constructor (Mercedes > Haas) increase predicted probability of top-10 finish; logistic regression outputs probabilities in [0, 1].

**Expected baseline performance vs docent floor:**

- Grid-rule docent baseline: Brier = 0.208 on test
- Calibrated docent model: Brier = 0.132 on test, ROC-AUC = 0.892
- Our team's best baseline expected to land near: Brier = 0.140–0.155 (modest improvement over grid-rule, calibration-aware)

---

## 4. What-If Comparison Plan

**Strategy variables we will vary:**

- [X] `grid_position`
- [X] `constructor_tier`
- [X] `driver_experience`
- [X] `drs_active_laps` (DRS activation window)
- [ ] Other: TBD (e.g., pit-stop undercuts, safety car risk)

**Concrete scenarios to compare (at least two, with specific values):**

> Scenario A: Mid-field team driver (constructor_tier=4) starting from P12 (outside points threshold) — predicted P(top-10) ≈ 0.15–0.25
> Scenario B: Same driver, same team, starting from P8 (inside points zone, advanced via qualifying gain) — predicted P(top-10) ≈ 0.68–0.82

**Decision metric for the comparison:**

> Difference in calibrated P(is_top10) between Scenario A and B, with confidence interval derived from logistic regression uncertainty quantification and bootstrap resampling on calibration set (season 2022).

---

## 5. Limitations Acknowledgment

**Five known dataset limitations are documented in the Capstone Brief. Which TWO most affect our team's specific approach?**

Limitation #1 we acknowledge: **Imbalanced class distribution** — top-10 finishes represent ~40–50% of outcomes; bottom-20 far more common in midfield

> Why it matters for our approach (1 sentence): Our baseline's Brier score (0.140–0.155) is limited by class imbalance; a model predicting "always 0" would still achieve Brier ≈ 0.45, so we must carefully weight false negatives (missing a genuine top-10 finisher) vs. false positives.

Limitation #2 we acknowledge: **Year-to-year regulation changes** — 2022 (calibration) differs from 2023–2024 (test); new cost cap, aerodynamic rules, tire compounds shift team pecking order

> Why it matters for our approach (1 sentence): Our calibration mapping (trained on 2022) may not generalize to 2023–2024 if team performance rankings shift significantly; we must validate calibration curve on early 2023 races before trusting predictions for mid-season strategy.

---

## 6. Experiment Plan for Hito 1

**Three experiments we will run between today and Wednesday 16:20:**

1. **Experiment 1: Baseline calibration curve** — Fit logistic regression (grid_position + constructor_tier) on train set (2019–2021), then apply isotonic calibration on calibration set (2022). Measure Brier Score on 2023–2024 test. Success = achieve Brier ≤ 0.155 (beating the grid-rule docent floor of 0.208).
2. **Experiment 2: Non-linear ensemble comparison** — Train Random Forest classifier on same features (grid, constructor, driver experience). Compare ROC-AUC and Brier Score vs. logistic regression on test set. Success = Random Forest captures non-linear decision boundaries (e.g., grid position thresholds like P10→P11 point boundary) and improves ROC-AUC by >0.05.
3. **Experiment 3: Cross-temporal calibration validation** — Fit calibration on season 2022, apply to early-season 2023 races (Rounds 1–3), and track whether predicted P(top-10) aligns with actual outcomes. Success = calibration curve remains stable (no >0.10 probability shift across years), confirming the mapping generalizes despite 2022→2023 rule changes.

---

## 7. Team Workflow

**Who is doing what between now and Wednesday?**

| Member           | Owns                                             | Branch / file in repo |
| ---------------- | ------------------------------------------------ | --------------------- |
| Martin Yungue    | Logistic regression baseline + calibration curve | `Capstone/Hito_1`   |
| Benjamin Bennett | Random Forest ensemble + ROC-AUC validation      | `Capstone/Hito_1`   |
| Group 20         | Integration + cross-temporal validation          | `Capstone/Hito_1`   |

**When does each member commit by?** (We need at least one commit per member per day Tue and Wed.)

- **Tuesday 16:00:** Baseline logistic regression fitted on 2019–2021, calibration on 2022 (Brier score logged)
- **Wednesday 10:00:** Random Forest trained, ROC-AUC comparison completed
- **Wednesday 16:00:** Final cross-temporal validation run on 2023–2024 test, all results merged to main

---

## 8. Critique Received in Pair Review

> *Filled during Block 5 (15:45–16:05) after the partner team reviews this sheet.*

**Reviewing team:** 5

**Concrete critique we received:**


**How we will address this critique by Wednesday:**


---

## Self-Check Before Committing

Before you push this to GitHub, verify:

- [X] Decision context is one sentence, not a paragraph
- [X] Target says exactly `is_top10` (not "Top-10" or "P(top10)")
- [X] Temporal split shows three blocks: 2019–2021 / 2022 / 2023–2024
- [X] Baseline is described in code-realistic terms (we could implement it)
- [X] What-if scenarios have specific feature values, not generic words
- [X] At least 2 of the 5 limitations are acknowledged with consequence
- [ ] PROMPTS.md exists in the repo (even if empty for now — will be populated by Wednesday)
