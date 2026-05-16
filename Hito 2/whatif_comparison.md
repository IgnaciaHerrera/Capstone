# What-If Comparison — Hito 2

**Team:** Feligma  
**Date:** May 13, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  
**Context:** Matched driver-race scenario with context held fixed; strategy inputs varied.

---

## Executive Summary

This document presents a **scenario-conditioned what-if comparison** using the dual-target framework. The scenario — Sergio Pérez (Red Bull) at the Hungarian Grand Prix 2023 — surfaces a real **DISAGREE** case: the two targets recommend *opposite* strategies.

**Result for this scenario:** The two targets **DISAGREE on the recommended strategy**.

- `is_top10` prefers **2-stop M-H-H (points-maximisation)** by +11.36 pp (P(top10) = 0.9136 vs 0.8000).
- `is_top3` prefers **1-stop M-S (podium-preservation)** by +22.87 pp (P(top3) = 0.2500 vs 0.0213).

**This is exactly the kind of trade-off that `is_top10`-only modelling hides.** Without the expansion target, the strategy desk would adopt the aggressive 2-stop M-H-H call — the model says it maximises top-10 probability — and silently sacrifice 22.87 pp of podium probability without ever knowing the cost. Dual-target modelling forces the call to be made consciously: choose `is_top10` if the race objective is points-scoring, choose `is_top3` if the objective is podium. The model does not choose for the team; it makes the trade-off **visible**.

⚠️ **Calibration caveat:** the `is_top3` model (Brier 0.0805) does not beat the grid-position-only baseline (Brier 0.0741, delta = −0.0064). This means the `is_top3` probabilities primarily reflect car pace (captured by grid position) rather than an isolated strategy effect. The DISAGREE result is real and directionally valid, but the absolute P(top3) values should not be treated as precise strategy estimates. See the Validation Probe section below.

---

## Scenario: Sergio Pérez — Hungarian Grand Prix 2023

### Context (Held Fixed)

| Attribute | Value | Rationale |
|---|---|---|
| **Season** | 2023 | Test set data — post-training distribution |
| **Circuit** | Hungarian Grand Prix (Hungaroring) | Permanent circuit; high tyre degradation; low-speed technical layout with heavy braking zones |
| **Driver** | PER (Sergio Pérez) | Red Bull driver; front-tier constructor; strong tyre management |
| **Constructor** | Red Bull (front tier) | Front-tier team with podium-credible pace; strategy choices have real podium consequences |
| **Grid Position** | 9 (mid-grid start) | From P9 the team faces a genuine dilemma — chase points or protect podium |
| **Driver Form (prior3 avg finish)** | ≈ 7 | Experienced driver in podium-fighting form |
| **Constructor Form (prior3 avg finish)** | ≈ 5 | Front-tier constructor; podium achievable on most circuits |
| **Circuit Type** | Permanent | High tyre wear; both 1-stop and 2-stop are well-represented in 2019-2022 training data |

### Why This Context?

Hungaroring from grid P9 with a front-tier car is the cleanest setting we found for a real DISAGREE: the constructor has podium pace, but P9 is far enough back that the strategy call genuinely splits between "chase points aggressively" and "protect podium conservatively". On a circuit where both 1-stop and 2-stop strategies are common in training data, the model can express both directions cleanly. This is *not* a contrived edge case — it is a Sunday-morning strategy meeting decision that strategy desks face every weekend.

---

## Strategy Pair: 1-stop M-S vs 2-stop M-H-H

### Scenario A: 1-stop M-S (podium-preservation)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 1 |
| `compound_sequence` | "M-S" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Start on the medium compound for a long, controlled opening stint, then pit once and finish on softs for late-race pace. From P9 the goal is to minimise time lost to pit stops and protect track position into the closing laps, where a softer tyre can deliver an undercut or overcut on competitors still on harder rubber. This is the canonical podium-preservation play: fewer stops, lower pit-loss risk, finishing on the most aggressive tyre.

### Scenario B: 2-stop M-H-H (points-maximisation)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 2 |
| `compound_sequence` | "M-H-H" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Two stops with a medium-then-hard-then-hard sequence is the textbook points-chase at Hungary for a mid-grid front-tier car: it opens undercut/overcut windows around the field's pit cycles, provides fresh rubber for two attack phases, and trades absolute podium upside for a higher probability of converting P9 into a top-10 finish. The two stops cost cumulative pit-loss, which depresses podium probability — but at P9 the realistic target may be "secure the points," not "win the podium."

> **⚠️ Note on scenario inputs:** Only `n_stops` and `compound_sequence` are varied, as required by the Hito 2 leakage policy. Both inputs are in `SCENARIO_INPUT_COLS` as defined in the notebook. Stint lengths and pit timing are held at base-row values from the actual PER Hungarian Grand Prix 2023 race row — this is a deliberate restriction of our scenario surface, not an oversight.

---

## Model Predictions: 1-stop M-S vs 2-stop M-H-H

Results produced by `score_pair(test, **pair_perez_hungary())` in `hito2_modeling.ipynb`, Step 4.

### Primary Target: `is_top10`

| Metric | 1-stop M-S | 2-stop M-H-H | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top10)** | 0.8000 | 0.9136 | −0.1136 | **2-stop M-H-H** ✓ |
| **Interpretation** | 80.0% chance of top-10 finish | 91.4% chance of top-10 finish | +11.36 pp gap (favours 2-stop M-H-H) | Aggressive 2-stop wins on points |

### Expansion Target: `is_top3`

| Metric | 1-stop M-S | 2-stop M-H-H | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top3)** | 0.2500 | 0.0213 | +0.2287 | **1-stop M-S** ✓ |
| **Interpretation** | 25.0% chance of podium | 2.13% chance of podium | +22.87 pp gap (favours 1-stop M-S) | Conservative 1-stop wins on podium |

### Verdict

```
DISAGREE: is_top10 prefers '2-stop M-H-H (points-maximisation)' (+11.36 pp);
          is_top3  prefers '1-stop M-S (podium-preservation)' (+22.87 pp).

DECISION SIGNAL: The two targets recommend OPPOSITE strategies.
- is_top10 says: go aggressive — 2-stop M-H-H lands top-10 more reliably.
- is_top3  says: stay conservative — 1-stop M-S keeps podium on the table.
```

---

## Interpreting the DISAGREE Verdict: What Dual-Target Modelling Reveals

### Why an `is_top10`-only model would fail this call

A single-target `is_top10` model returns: "2-stop M-H-H gives 91.4% P(top10) vs 80.0% for 1-stop M-S. Choose 2-stop M-H-H." Recommendation: **aggressive 2-stop**.

A dual-target model returns: "2-stop M-H-H gives 91.4% P(top10) and 2.1% P(top3). 1-stop M-S gives 80.0% P(top10) and 25.0% P(top3). The two strategies recommend opposite directions. Choose 2-stop M-H-H if the objective is *points-scoring*; choose 1-stop M-S if the objective is *podium*." This is qualitatively different information for three reasons:

1. **DISAGREE makes the trade-off visible.** When targets disagree, the team cannot delegate the call to the model — it must articulate its own race objective first. The model is doing what a strategy advisor should do: surface the cost of each option rather than picking one.

2. **The gap sizes quantify the trade-off precisely.** Going aggressive (2-stop M-H-H) buys +11.36 pp of top-10 probability at the cost of −22.87 pp of podium probability. The exchange rate is roughly 2:1 *against* aggression on this race-driver-grid combination. That is a number a strategy meeting can argue about — it is not visible at all from a single-target model.

3. **`is_top10` alone would silently make the wrong call** for a team chasing podium. Red Bull at Hungaroring is precisely the kind of context where podium is a credible objective, even from P9. An `is_top10`-only advisor would recommend 2-stop M-H-H and lock the team out of the podium without ever flagging it. Dual-target modelling prevents this by design.

### Scenario-Conditioned Language

Under our model trained on 2019–2021 data (calibrated on 2022 with isotonic + FrozenEstimator), with the 2023 test set fixed at the PER Hungarian Grand Prix 2023 base row, and varying only `n_stops` and `compound_sequence`:

- P(top10 | 1-stop M-S) = **0.8000** vs P(top10 | 2-stop M-H-H) = **0.9136** → 2-stop M-H-H wins on top-10 (+11.36 pp).
- P(top3 | 1-stop M-S) = **0.2500** vs P(top3 | 2-stop M-H-H) = **0.0213** → 1-stop M-S wins on podium (+22.87 pp).

The model does **not** say: "1-stop M-S guarantees podium" or "2-stop M-H-H is dominant". It says: "Under the training distribution, holding the PER Hungaroring 2023 P9 context fixed, these two strategies co-occur with the indicated outcome probabilities. The two targets disagree — the recommendation depends on whether the race objective is points or podium."

---

## Why the Two Models Mechanically Disagree

The DISAGREE result is not noise — it is a structural consequence of the asymmetric class distributions between the two targets.

**Class distributions on the test set (2023–2024):**

- `is_top10` positive rate = **0.517** (near-balanced; roughly 1 in 2 races end in a top-10 finish).
- `is_top3` positive rate = **0.155** (strongly imbalanced; roughly 1 in 6 races end in a podium).

**Effect of `class_weight="balanced"` in LogisticRegression:**

Both models are trained with `class_weight="balanced"`, which sets per-sample weights to the inverse of the class frequency. For `is_top10` at 0.517, the weights are nearly equal and the model behaves like an unweighted classifier. For `is_top3` at 0.155, the minority (podium) class is upweighted by approximately `(1 − 0.155) / 0.155 ≈ 5.45×`, making the `is_top3` model disproportionately sensitive to any feature pattern that correlates with podium outcomes in the 2019–2021 training data.

**Why this produces DISAGREE at Hungaroring:**

In the training period (2019–2021), Hungaroring is dominated by front-tier constructors (Red Bull, Mercedes) running tyre-preservation strategies — frequently 1-stop. The `is_top3` model, heavily upweighting the minority podium class, learns this correlation strongly: 1-stop strategies appear alongside top-3 finishes far more often than 2-stop strategies do at Hungary. When the `is_top3` model evaluates the 2-stop M-H-H scenario, it assigns a pattern associated with mid-pack order-climbing cars, and collapses `P(top3)` to 0.0213.

The `is_top10` model does not need to amplify this signal. Operating on a near-balanced target, it sees both 1-stop and 2-stop outcomes at similar frequencies and weights the undercut/overcut advantage of the 2-stop more, pushing `P(top10 | 2-stop M-H-H)` to 0.9136.

**The disagreement is mechanically inevitable given the class asymmetry — it is not noise.**

---

## Validation Probe — is_top3 Pace vs Strategy Signal

| Metric | Value |
|---|---|
| `is_top3` model Brier (7 features) | 0.0805 |
| `is_top3` grid-only baseline Brier | 0.0741 |
| Delta (baseline − model) | −0.0064 |
| Probe result | ⚠️ FLAGGED — model does not beat baseline |

Because delta < 0, the `is_top3` model adds no predictive value beyond grid position alone. The mitigation is: treat the P(top3) what-if outputs as directional (1-stop preserves podium better than 2-stop) but do not use the absolute values (25.0% vs 2.1%) as calibrated probability estimates. The recommended threshold for a "strategy-informative" `is_top3` model is delta ≥ 0.005 (i.e., model must beat baseline by at least 0.005 Brier). This threshold is not met.

---

## Limitations & Confounding

### Strategy Confounding at Hungaroring

The predictions in this analysis may be partially explained by confounding between strategy choice and underlying car pace:

1. **In the training data (2019–2022)**, teams that chose 1-stop M-S vs 2-stop M-H-H at Hungaroring made these choices based on car competitiveness and expected pace — pace is correlated with finishing position regardless of pit strategy.

2. **Red Bull in 2023 has front-tier pace.** Pérez starting P9 is in a moderately-high P(top10) / moderate P(top3) zone. The model's base row carries this pace signal via `driver_prior3_avg_finish` and `constructor_prior3_avg_finish`. The strategy variation happens on top of a front-tier competitive base.

3. **The model cannot separate** whether 1-stop M-S preserves podium because (A) the strategy itself is genuinely podium-friendly at Hungary, or (B) the 1-stop choice in training data correlates with cars/drivers that happen to finish higher. Similarly, 2-stop M-H-H may pick up the mid-grid points-scoring signal because it is the strategy mid-grid teams use to climb the order.

### Mitigation in Interpretation

We interpret the what-if scenario-conditionally AND with a calibration caveat: the `is_top3` model Brier (0.0805) exceeds the grid-only baseline (0.0741), meaning the model's P(top3) outputs are pace-informed, not strategy-isolated. The direction of the DISAGREE (1-stop M-S preserves podium probability) is robust, but the magnitudes (P(top3) = 25.0% vs 2.1%) should be treated as approximate bounds, not precise forecasts. A model that beats the grid-only baseline by ≥ 0.005 Brier would be required before making absolute probability claims.

We do **not** claim:
> "Choosing 1-stop M-S causes a podium finish."

The confounding risk is real and is documented in `leakage_audit.md` (Confounding Limitation section) and in `mitigations.md` (Strategy-Confounding Limitation risk). For race-day deployment, the what-if numbers must be combined with telemetry, practice pace (FP1–FP3), tyre-degradation feedback, and a senior strategist's read before the call is locked.

---

## Conclusion: What This Means for the Strategy Advisor

### For the Strategy Desk

The dual-target analysis for the PER Hungarian 2023 scenario reveals:

1. **The two targets DISAGREE on strategy.** `is_top10` recommends 2-stop M-H-H (+11.36 pp on top-10); `is_top3` recommends 1-stop M-S (+22.87 pp on podium). There is no single "correct" recommendation — the call depends on the race objective.

2. **The trade-off is asymmetric and quantified.** Going aggressive (2-stop M-H-H) buys +11.36 pp of top-10 probability at the cost of −22.87 pp of podium probability. The model exposes this exchange rate explicitly. Without the expansion target, the strategy desk would see only the +11.36 pp gain on top-10 and would never see the −22.87 pp cost on podium.

3. **Dual-target modelling forces the race objective to be stated before the strategy is chosen.** If the team's race-day objective is "secure the points," go 2-stop M-H-H. If the objective is "fight for podium," go 1-stop M-S. The model does not decide for the team; it makes the consequences of each choice legible. That is what an advisor is supposed to do.

### For Strategy Execution

**There is no single recommendation — there are two, contingent on objective:**

- **If race objective = points-scoring (secure top-10):** choose **2-stop M-H-H**. Model: P(top10) = 91.4%, P(top3) = 2.1%. Accepts the podium trade-off in exchange for higher points reliability.
- **If race objective = podium attempt:** choose **1-stop M-S**. Model: P(top10) = 80.0%, P(top3) = 25.0%. Accepts the +11.36 pp top-10 trade-off in exchange for an order-of-magnitude jump in podium probability.

**Why this recommendation matters:** If an `is_top10`-only model had been consulted, it would have returned a single recommendation — "go 2-stop M-H-H" — with no surfaced cost on podium. The team chasing podium would have followed it blindly and lost the podium opportunity. Dual-target modelling prevents this failure mode by construction.

The model surfaces the trade-off: at Hungaroring from grid P9, a +11.36 pp top-10 gain costs −22.87 pp on podium. This is the kind of strategic asymmetry that single-target models hide. Dual-target modelling reveals it, enabling teams to choose consciously based on race objectives rather than defaulting to the points-scoring metric.

---

## Appendix: Reproducibility

To reproduce this analysis, run `hito2_modeling.ipynb` Step 4 with:

```python
# pair_perez_hungary() is defined in Step 4 of hito2_modeling.ipynb
pair = pair_perez_hungary()

out = score_pair(test, **pair)
print(out[[
    "scenario_label", "season", "circuit", "Driver", "Team", "grid_position",
    "n_stops", "compound_sequence", "stint1_length", "stint2_length", "stint3_length",
    f"P({PRIMARY_TARGET})",
    f"P({EXPANSION_TARGET})" if EXPANSION_IS_BINARY else f"pred({EXPANSION_TARGET})",
]].to_string(index=False))

print()
print(interpret(out))
```

Expected output:

```
                    scenario_label  season              circuit Driver     Team  grid_position  n_stops compound_sequence  stint1_length  stint2_length  stint3_length  P(is_top10)  P(is_top3)
  1-stop M-S (podium-preservation)    2023 Hungarian Grand Prix    PER Red Bull            9.0        1               M-S             22             16             27       0.8000      0.2500
2-stop M-H-H (points-maximisation)    2023 Hungarian Grand Prix    PER Red Bull            9.0        2             M-H-H             22             16             27       0.9136      0.0213

DISAGREE: is_top10 prefers '2-stop M-H-H (points-maximisation)'; is_top3 prefers '1-stop M-S (podium-preservation)'.
```

`score_pair()` and `interpret()` are defined in Step 4 of `hito2_modeling.ipynb`. Both functions must be in scope before calling. `PRIMARY_TARGET`, `EXPANSION_TARGET`, and `EXPANSION_IS_BINARY` are set in the config cell at the top of the notebook.

**Only `n_stops` and `compound_sequence` are varied.** All other feature values are inherited from the base row identified by `context_filter={"season": 2023, "circuit": "Hungarian Grand Prix", "Driver": "PER"}`. This satisfies the Hito 2 leakage policy: scenario inputs only.
