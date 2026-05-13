# What-If Comparison — Hito 2

**Team:** Feligma  
**Date:** May 12, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  
**Context:** Matched driver-race scenarios with fixed context, strategy inputs varied

---

## Executive Summary

This document presents a **scenario-conditioned what-if comparison** using the dual-target framework. The primary scenario — Sergio Pérez (Red Bull) at the Hungarian Grand Prix 2023 — demonstrates the value of modeling both `is_top10` and `is_top3` simultaneously, especially when targets **disagree**.

**Result for this scenario:** The two targets **fundamentally disagree on strategy**. The `is_top10` model prefers 2-stop (+5.0 pp, 92.8% vs 87.8%), while the `is_top3` model strongly prefers 1-stop (+22.9 pp, 25.0% vs 2.1%). This reveals an asymmetric trade-off: 2-stop improves points-scoring chances slightly but sacrifices podium chances catastrophically. A `is_top10`-only model would recommend 2-stop without hesitation, missing the podium downside entirely.

**The strategic value of dual-target modeling:** In disagreement scenarios, teams must make explicit trade-offs between competing objectives. Is the goal to maximize points (choose 2-stop, +5 pp top-10) or to protect podium potential (choose 1-stop, +22.9 pp top-3)? Dual-target modeling surfaces this trade-off; single-target modeling hides it. From grid P9 at Hungaroring, the choice becomes clear: if podium is possible, sacrificing 22.9 pp of podium probability to gain 5 pp of top-10 probability is a poor exchange.

---

## Scenario: Sergio Pérez — Hungarian Grand Prix 2023

### Context (Held Fixed)

| Attribute | Value | Rationale |
|---|---|---|
| **Season** | 2023 | Test set data — post-training distribution |
| **Circuit** | Hungarian Grand Prix (Hungaroring) | Permanent circuit; high tire degradation; low-speed technical circuit with heavy braking zones |
| **Driver** | PER (Sergio Pérez) | Red Bull driver; front-tier constructor; competitive pace in midfield positions |
| **Constructor** | Red Bull (Tier 1 front) | Competitive front-tier team; strategy and tire management are critical tactical levers |
| **Grid Position** | 9 (midfield start) | From P9, podium is possible but requires top-tier pit strategy execution |
| **Driver Form** | prior3 avg finish ≈ 7th | Experienced driver; strong in tire management and strategic decision-making |
| **Constructor Form** | prior3 avg finish ≈ 5th | Front-tier constructor; podium achievable on most circuits |
| **Circuit Type** | Permanent | Low-speed technical circuit; high tire wear; two-stop strategies common in training data |

### Why This Context?

Hungaroring from grid P9 creates a scenario where the strategy choice reveals fundamental disagreement between top-10 and top-3 objectives. The track's high tire degradation and complex pit strategy dynamics (typical two-stop turf in our training data) make 2-stop competitive for points-scoring. Yet from midfield grid position, podium preservation via track position is valuable. This is a natural DISAGREE scenario: 2-stop is marginally better for top-10 (+5.0 pp) but dramatically worse for podium (-22.9 pp). This choice reveals what dual-target modeling adds: the ability to surface strategic trade-offs that `is_top10` alone would hide.

---

## Strategy Pair: One-Stop Tire Conservative (M-H) vs Two-Stop Tactical (H-M-M)

### Scenario A: One-Stop Conservative (M-H)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 1 |
| `compound_sequence` | "M-H" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Start on medium, pit once for hard (or stay medium long, pit for hard later). Single pit minimizes pit loss and enables controlled tire warm-up. Preserves track position; classic "manage tire life" strategy from midfield that avoids the risk of being undercut or losing net position to traffic during multi-stop sequences.

### Scenario B: Two-Stop Tactical (H-M-M)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 2 |
| `compound_sequence` | "H-M-M" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Start on hard, pit for medium, pit again for medium (or adjust final compound). Two stops enable tactical undercut/overcut options, fresh rubber in critical stints, and flexibility to respond to competitors' pit timings. At Hungaroring with high tire wear, this is a competitive points-scoring strategy. However, two stops incur cumulative pit loss (≈2× pit stop penalties vs 1×), risking net position loss.

> **⚠️ Note on scenario inputs:** Only `n_stops` and `compound_sequence` are varied, as required by the Hito 2 leakage policy. Both inputs are in `SCENARIO_INPUT_COLS` as defined in the notebook. Stint lengths and pit timing are held at base-row values from the Hungarian Grand Prix 2023 PER race — this is a restriction of our feature set, not an oversight.

---

## Model Predictions: One-Stop vs Two-Stop

Results produced by `score_pair(test, **pair_perez_hungary())` in `hito2_modeling.ipynb`, Step 4.

### Primary Target: `is_top10`

| Metric | One-Stop (M-H) | Two-Stop (H-M-M) | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top10)** | 0.878 | 0.928 | −0.050 | **Two-stop** ✓ |
| **Interpretation** | 87.8% chance of top-10 finish | 92.8% chance of top-10 finish | 5.0 pp gap (favors 2-stop) | 2-stop improves points |

### Expansion Target: `is_top3`

| Metric | One-Stop (M-H) | Two-Stop (H-M-M) | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top3)** | 0.250 | 0.021 | +0.229 | **One-stop** ✓ |
| **Interpretation** | 25.0% chance of podium | 2.1% chance of podium | 22.9 pp gap (strongly favors 1-stop) | 1-stop protects podium |

### Verdict

```
DISAGREE: is_top10 prefers '2-stop H-M-M (points aggressive)'; is_top3 prefers '1-stop M-H (podium conservative)'.
```

---

## Interpreting the Agreement with Asymmetric Strength: What Dual-Target Modeling Reveals

### Why Asymmetric Agreement Margins Are a Finding

A single-target `is_top10` model returns: "2-stop is preferred by 5.0 pp." Recommendation: use 2-stop.

A dual-target model returns: "2-stop is preferred by 5.0 pp on top-10, BUT 1-stop is preferred by 22.9 pp on podium." This is qualitatively different information for three reasons:

1. **DISAGREEMENT reveals the true choice structure.** The targets point in opposite directions. The `is_top10` model says 2-stop is better (92.8% vs 87.8%); the `is_top3` model says 1-stop is dramatically better (25.0% vs 2.1%). This is not a minor disagreement — it's a fundamental strategic fork. A `is_top10`-only model would choose 2-stop without hesitation, missing the fact that 2-stop tanks podium probability by 22.9 pp.

2. **The gap sizes reveal the stakes asymmetry.** For top-10, the gap is small (5.0 pp). For podium, the gap is massive (22.9 pp). This tells the strategy desk: "Gaining 5 pp in top-10 probability costs you 22.9 pp in podium probability. This is a terrible trade." If podium is within reach (25% for 1-stop), sacrificing it for a marginal top-10 gain is poor risk management.

3. **DISAGREE scenarios enable explicit trade-off decisions.** From grid P9 at Hungaroring, the team must ask: "What is our race goal?" If it's points-scoring at any cost, choose 2-stop (+5 pp top-10). If it's podium protection, choose 1-stop (preserve 25% podium odds vs lose to 2% with 2-stop). Dual-target modeling surfaces the trade-off; `is_top10` alone conceals it. This is the core value of expansion targets.

### Scenario-Conditioned Language

Under our model trained on 2019–2021 data (calibrated on 2022), with the 2023 test set fixed at the PER Hungarian Grand Prix 2023 base row, and varying only `n_stops` and `compound_sequence`:

- P(top10 | 1-stop M-H) = **0.878** vs P(top10 | 2-stop H-M-M) = **0.928** → 5.0 pp gap (2-stop preferred)
- P(top3 | 1-stop M-H) = **0.250** vs P(top3 | 2-stop H-M-M) = **0.021** → 22.9 pp gap (1-stop strongly preferred)

The model does **not** say: "1-stop guarantees podium" or "2-stop guarantees points." It says: "Under the training distribution, holding Pérez at Hungaroring 2023 fixed at grid P9, two strategies co-occur with different outcomes on each target. The choice between them depends on the team's race objective."

---

## When Targets Would Disagree: The Trade-Off Structure

The Hito 2 requirement is to demonstrate what dual-target analysis adds beyond `is_top10`. The Sainz-Monza scenario shows agreement. The natural follow-up is: **under what conditions would the targets disagree?**

### Structural Conditions for Disagreement

Based on the model behavior and training data:

| Condition | Expected Effect |
|---|---|
| **Lower-tier constructor (Tier 3), starting P13–P15** | 2-stop may raise P(top10) by aggressive early undercut, but fail to consistently reach top-3 → potential AGREE in opposite direction (2-stop preferred on both) |
| **Street circuit (Singapore, Monaco) — high tire deg** | Multiple stops correlate with points-scoring outcomes historically; but podium from P10+ is rare regardless → likely AGREE (one target may be near-floor) |
| **Pure midfield driver, competitive day** | 1-stop conservative may land P8–P9 safely; 2-stop aggressive may land P4–P5 or P12 → wider P(top3) gap with narrower P(top10) gap → **DISAGREE candidate** |
| **Wet or mixed conditions** | Strategy correlation breaks down; model may assign similar probabilities to both scenarios under either target → AGREE (near-tied) |

A hypothetical DISAGREE scenario would look like:

> Driver at P10 on grid, Tier 2 constructor, permanent circuit. 1-stop → P(top10) = 0.62, P(top3) = 0.04. 2-stop → P(top10) = 0.55, P(top3) = 0.09. `is_top10` prefers 1-stop; `is_top3` prefers 2-stop.

In this case the trade-off is real: the team must choose between maximizing points-scoring probability and maximizing podium probability. That choice requires an explicit team objective. `is_top10` alone would recommend 1-stop unconditionally, missing the 2-stop's podium upside entirely.

---

## Limitations & Confounding

### Strategy Confounding at Monza

The predictions in this analysis may be partially explained by confounding between strategy choice and underlying car pace:

1. **In the training data (2019–2022)**, teams that chose 2-stop at Monza often did so because they had the pace to attack — and pace is correlated with finishing position regardless of strategy.

2. **Ferrari in 2024 has front-row pace.** Sainz starting P5 at Monza is already in a high-P(top10), moderate-P(top3) zone. The model's base row carries this pace signal in `prior_avg_finish_driver` and `prior_avg_finish_constructor`. The strategy variation happens on top of a high-performing base.

3. **The model cannot separate** whether 1-stop improves outcomes because (A) the strategy itself is better for this car at this circuit, or (B) drivers with good pace happen to choose 1-stop at Monza more often in our training data.

### Mitigation in Interpretation

We interpret the what-if scenario-conditionally:
> "Holding driver pace and team capability fixed at the PER Hungarian Grand Prix 2023 base-row values, and varying only strategy inputs, the model predicts these probabilities."

We do **not** claim:
> "Choosing 1-stop causes a podium finish."

The confounding risk exists: teams with stronger pace and tire management may choose 1-stop (track position preservation) more often in training data. The model captures this correlation but cannot definitively separate strategy effect from underlying pace. A driver with poor tire deg might need 2-stop regardless of grid position. This is why the strategy advisor must be combined with telemetry and practice-pace data before race-day deployment.

---

## Conclusion: What This Means for the Strategy Advisor

### For the Strategy Desk

The dual-target analysis for the PER Hungarian 2023 scenario reveals:

1. **The two targets fundamentally disagree on strategy.** `is_top10` prefers 2-stop by 5.0 pp. `is_top3` prefers 1-stop by 22.9 pp. This is not a tie-breaking scenario; it's a genuine strategic fork.

2. **The trade-off is asymmetric and unfavorable to 2-stop if podium matters.** Choosing 2-stop gains +5.0 pp in top-10 but costs −22.9 pp in podium. From grid P9, if podium is within reach (25% for 1-stop, 2% for 2-stop), this is a terrible exchange: lose 23 pp of podium probability to gain 5 pp of top-10 probability. The risk-reward is upside-down.

3. **Dual-target modeling enables explicit trade-off decisions.** Under `is_top10` alone, the choice is clear: use 2-stop (+5.0 pp). Dual-target analysis reveals: "Yes, 2-stop improves top-10, but it destroys podium chances. If your race goal is podium, this is non-negotiable; choose 1-stop and protect the 25% podium odds. If your race goal is pure points-scoring regardless of podium, choose 2-stop and accept the 23 pp podium loss." This is actionable decision support.

### For Strategy Execution

If the race-day objective is **"maximize points scoring,"** use `is_top10` → 2-stop H-M-M is recommended with +5.0 pp margin (92.8% vs 87.8%). If the race-day objective is **"protect podium potential,"** use `is_top3` → 1-stop M-H is non-negotiable with +22.9 pp margin (25.0% vs 2.1%).

The model surfaces the trade-off: at Hungaroring from grid P9, a 5 pp improvement in top-10 probability costs 23 pp in podium probability. This is the kind of strategic asymmetry that single-target models hide. Dual-target modeling reveals it, enabling teams to choose consciously based on race objectives rather than defaulting to the points-scoring metric.

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
DISAGREE: is_top10 prefers '2-stop H-M-M (points aggressive)'; is_top3 prefers '1-stop M-H (podium conservative)'.
```

Or more verbosely:

```
                     scenario_label  season              circuit Driver     Team  grid_position  n_stops compound_sequence  stint1_length  stint2_length  stint3_length  P(is_top10)  P(is_top3)
1-stop M-H (podium conservative)    2023 Hungarian Grand Prix    PER Red Bull            9.0        1               M-H             22             16             27     0.877564    0.250000
2-stop H-M-M (points aggressive)    2023 Hungarian Grand Prix    PER Red Bull            9.0        2             H-M-M             22             16             27     0.927564    0.021277

DISAGREE: is_top10 prefers '2-stop H-M-M (points aggressive)'; is_top3 prefers '1-stop M-H (podium conservative)'.
```

`score_pair()` and `interpret()` are defined in Step 4 of `hito2_modeling.ipynb`. Both functions are required to be in scope before calling. `PRIMARY_TARGET`, `EXPANSION_TARGET`, and `EXPANSION_IS_BINARY` are set in the config cell at the top of the notebook.

**Only `n_stops` and `compound_sequence` are varied.** All other feature values are inherited from the base row identified by `context_filter={"season": 2023, "circuit": "Hungarian Grand Prix", "Driver": "PER"}`. This satisfies the Hito 2 leakage policy: scenario inputs only.
