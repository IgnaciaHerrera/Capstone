# Capstone Project Brief — F1 Race Strategy Advisor

## Problem

Formula 1 teams do not only ask whether a driver will finish in the Top 10. During a race weekend, the more useful question is:

> **If we choose a different pit-stop strategy, how does the expected race outcome change?**

Your team will build a **Race Strategy Advisor**: a reproducible ML system that compares strategy scenarios and translates model output into a recommendation for a race strategy desk.

---

## Required Scope

All teams must complete the race-level advisor (Layer 1). Strong teams may attempt Layer 2.

### Layer 1 — Race-Level Strategy Advisor

#### Dataset
- **f1_strategy_race_level.csv** (2,447 rows · seasons 2019–2024 · 1 row per driver-race)

#### Prediction Unit
One driver in one race.

#### Target Progression (Mandatory Across Milestones)

| Milestone | Target Requirement |
|-----------|-------------------|
| Milestone 1 | All teams use `is_top10`. Uniform target across the cohort enables direct comparison against the docent baseline. |
| Milestone 2 | Mandatory expansion: add at least one additional target from `{is_top5, is_top3, finish_position, points}` and compare decision-value across the two targets. |
| Final Report | Free choice — teams may stay with their Milestone 2 target pair or expand further if it strengthens the strategy recommendation. |

#### Minimum Modeling Expectation

Build a leakage-aware, calibrated model that estimates P(finish_position ≤ k) and compares at least two strategy scenarios.

**Example scenario:**

For a midfield starter at Monza, compare a one-stop M-H strategy against a two-stop M-H-S strategy. Which one gives better expected Top-10 probability, and under what assumptions?

### Layer 2 — Stint-Level Degradation Simulator (Optional)

#### Dataset
- **f1_strategy_lap_level.csv** (119,908 rows · 1 row per driver-race-lap)

This is optional stretch work. Teams use lap-level data to model `lap_time_delta` or degradation as a function of tyre age, compound, stint phase, track status, and weather context.

---

## What Counts as a Good Recommendation

A good recommendation is not just "model A has higher accuracy." It should say:

1. What strategy you recommend.
2. What outcome metric supports it.
3. How large the expected benefit is.
4. Which assumptions could invalidate the recommendation.
5. How reliable/calibrated the model is.

---

## Required Analyses

Every team must include:

- [ ] A simple baseline.
- [ ] A trained model.
- [ ] Temporal validation by season.
- [ ] Calibration or probability-quality evaluation.
- [ ] Error analysis sliced by strategy type and circuit type.
- [ ] At least one what-if comparison.
- [ ] An explicit leakage/confounding discussion.
- [ ] PROMPTS.md documenting AI use.

### Recommended Metrics

- **Brier score**
- **Log loss**
- **Calibration curve**
- **ROC-AUC** or average precision (secondary metrics)
- **MAE** if modeling `finish_position`

---

## Docent Baseline (Floor Your Team Must Clear)

A docent baseline is provided. Your Milestone 1 model must match or beat it on the same target (`is_top10`) and the same temporal split, OR your framing document must justify why your alternative approach is better aligned with the strategy decision your team is supporting.

| Metric | Value |
|--------|-------|
| Target | `is_top10` |
| Train | seasons 2019–2021 |
| Calibration | season 2022 |
| Test | seasons 2023–2024 |
| Grid-rule baseline (Brier score on test) | 0.208 |
| Calibrated docent model (Brier score on test) | 0.132 |
| Calibrated docent model (ROC-AUC on test) | 0.892 |

> The grid-rule baseline is the easy floor. The calibrated docent model is the meaningful floor — beating it requires real work. Both are reported so your team can see what good looks like before you start.

---

## Leakage Rules

Strategy features such as `n_stops`, `compound_sequence`, and `stint_lengths` are post-race observations in the raw data. In any other context in this course, using them as predictors would be leakage.

**For this capstone they are allowed** because the product is a scenario comparison tool: you intentionally set those variables to ask "what if?" The model receives them as user-controlled scenario inputs, not as information that was magically known before the race. **Your framing document must declare this distinction explicitly** — it is a graded item.

### Data Usage Guidelines

Do not silently use race-incident columns such as `safety car` or `weather outcome` as if they were known before the race. If you use them, frame them as:

- Audit slices
- Scenario stress tests
- Limitations

---

## Known Data Limitations

The dataset is a recovery build assembled from existing course artifacts. Five limitations apply:

1. **Coverage starts in 2019**, not 2018, because the recovered lap-level artifact is 2019–2024.

2. **Qualifying position**: `qualifying_position` is a stand-in for `grid_position`; `qualifying_time_s` is empty. The column is kept for consistency but do not build a story around qualifying time. Treating qualifying as a real signal is a graded error.

3. **Safety car indicator**: `safety_car_periods` is a binary indicator per driver-race, not a full race-control interval count.

4. **Strategy features are observed post-race** (see Leakage Rules above). They are scenario inputs in this capstone, not pre-race signals.

5. **Strategy choice is not independent**: Strategy choice is not independent of car pace, driver, weather, and race incidents. Teams must discuss this confounding when they make recommendations.

> Recognizing and disclosing these limitations contributes to your "Identification of limitations" score (Dimension 3 of the rubric).

---

## Deliverables

### Milestone 1 — Problem Framing + Baseline

**Due:** Wed May 6, 16:20 CLT  
(in-class submission with TA observation — see the Milestone 1 Canvas assignment for the full submission policy and individual penalty matrix)

**Submit:**

- [ ] A 2–3 page framing document.
- [ ] A runnable baseline notebook with target = `is_top10`.
- [ ] Initial data quality notes that disclose the dataset limitations.
- [ ] Temporal validation plan (train ≤ 2022 / test ≥ 2023 is the course standard; the docent baseline uses 2019–2021 train / 2022 calibration / 2023–2024 test as a teaching variant).
- [ ] First what-if comparison plan.
- [ ] PROMPTS.md.

### Milestone 2 — Midpoint Model + Error Analysis

**Due:** Wed May 13, 16:20 CLT

**Submit:**

- [ ] Updated modeling notebook covering both targets (your Milestone 1 `is_top10` plus your mandatory expansion target).
- [ ] Baseline vs model comparison on both targets.
- [ ] Calibration / probability-quality analysis on both targets.
- [ ] Error analysis sliced by strategy type, circuit type, and at least one additional context.
- [ ] At least one what-if strategy comparison that uses the expanded target to surface a recommendation `is_top10` alone could not produce.
- [ ] Mitigation plan for the final report.
- [ ] Updated PROMPTS.md.

### Final Report + Repository

**Due:** Sun May 17, 23:59 CLT

**Submit a reproducible repo and an 8–12 page report.**

### Demo Day

**Date:** Mon May 18, in class

**Pitch Format:**
- 7 minutes recommendation + 5 minutes Q&A.
- Frame the presentation as a strategy recommendation to an F1 engineering audience.

---

*Last Updated: May 4, 2026*
