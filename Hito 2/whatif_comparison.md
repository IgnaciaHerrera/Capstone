# What-If Comparison — Hito 2

**Team:** Feligma  
**Date:** May 12, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  
**Context:** Matched driver-race scenarios with fixed context, strategy inputs varied

---

## Executive Summary

This document presents a **scenario-conditioned what-if comparison** using the dual-target framework. The primary scenario — Carlos Sainz (Ferrari) at the Italian Grand Prix 2024 — is used to demonstrate the value of modeling both `is_top10` and `is_top3` simultaneously.

**Result for this scenario:** The two targets **agree** that a one-stop conservative strategy (M-H) is preferred over a two-stop aggressive strategy (S-M-H) — both in terms of P(top10) and P(top3). This agreement is itself a finding: it amplifies confidence that 1-stop is the dominant choice for this specific driver-race context. Under a pure `is_top10` model, this confidence amplification is invisible.

**The strategic value of dual-target modeling:** Agreement across both targets strengthens a recommendation in a way that single-target analysis cannot. Conversely, disagreement between the targets — a case where 1-stop raises P(top10) but lowers P(top3) — would signal a genuine trade-off requiring an explicit team objective decision. Both outcomes (agreement and disagreement) are information; `is_top10` alone produces neither.

---

## Scenario: Carlos Sainz — Italian Grand Prix 2024

### Context (Held Fixed)

| Attribute | Value | Rationale |
|---|---|---|
| **Season** | 2024 | Test set data — post-training distribution |
| **Circuit** | Italian Grand Prix (Monza) | Permanent circuit; high-speed; classic strategy split circuit |
| **Driver** | SAI (Carlos Sainz) | Ferrari driver; competitive pace; not pure midfield |
| **Constructor** | Ferrari (Tier 1–2 boundary) | Competitive but not dominant; strategy decisions matter |
| **Grid Position** | 5 (base row value) | Front-midfield start; within top-10 contention |
| **Driver Form** | prior3 avg finish ≈ 6th | Consistent recent performance |
| **Constructor Form** | prior3 avg finish ≈ 5th | Ferrari near-front consistent pace |
| **Circuit Type** | Permanent | Low tire degradation relative to street circuits |

### Why This Context?

Monza is a classic one-stop vs two-stop decision circuit. Its low tire degradation and long straights create genuine tension between: (a) a conservative 1-stop strategy preserving track position, and (b) an aggressive 2-stop soft-compound strategy targeting early pace and overtaking. A driver at grid position 5 with Tier 1–2 pace has real choices here — this is not a context where strategy is predetermined by pace ceiling.

---

## Strategy Pair: One-Stop (Conservative) vs Two-Stop (Aggressive)

### Scenario A: One-Stop Conservative (M-H)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 1 |
| `compound_sequence` | "M-H" |

All other features held at base-row values from the Italian Grand Prix 2024 SAI context.

**Rationale:** Start on medium, switch to hard. Single pit minimizes time loss and traffic risk. Classic Monza "tire preservation" approach that relies on car pace at the front.

### Scenario B: Two-Stop Aggressive (S-M-H)

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 2 |
| `compound_sequence` | "S-M-H aggressive" |

All other features held at base-row values from the Italian Grand Prix 2024 SAI context.

**Rationale:** Start on soft, push early pace, manage two pit windows. Higher upside if pace is there; exposes to traffic and undercut/overcut timing risk. Historically chosen by teams attacking podium from behind.

> **⚠️ Note on scenario inputs:** Only `n_stops` and `compound_sequence` are varied, as required by the Hito 2 leakage policy. Both inputs are in `SCENARIO_INPUT_COLS` as defined in the notebook. Stint lengths and pit timing are held at base-row values — this is a restriction of our feature set, not an oversight.

---

## Model Predictions: One-Stop vs Two-Stop

Results produced by `score_pair(test, **pair_sainz_monza())` in `hito2_modeling.ipynb`, Step 4.

### Primary Target: `is_top10`

| Metric | One-Stop (M-H) | Two-Stop (S-M-H) | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top10)** | 0.956 | 0.835 | +0.121 | **One-stop** ✓ |
| **Interpretation** | 95.6% chance of top-10 finish | 83.5% chance of top-10 finish | 12.1 pp gap | 1-stop dominates |

### Expansion Target: `is_top3`

| Metric | One-Stop (M-H) | Two-Stop (S-M-H) | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top3)** | 0.559 | 0.466 | +0.093 | **One-stop** ✓ |
| **Interpretation** | 55.9% chance of podium | 46.6% chance of podium | 9.3 pp gap | 1-stop dominates |

### Verdict

```
AGREE: is_top10 prefers '1-stop M-H'; is_top3 prefers '1-stop M-H'.
```

---

## Interpreting the Agreement: What Dual-Target Modeling Adds

### Why Agreement Is a Finding, Not a Null Result

A single-target `is_top10` model returns: "1-stop preferred by 12.1 pp." Full stop.

A dual-target model returns: "1-stop preferred by 12.1 pp on top-10 AND by 9.3 pp on top-3." This is qualitatively different information for three reasons:

1. **No hidden trade-off.** The advisor can confidently recommend 1-stop without reservations about podium sacrifice. Under `is_top10` alone, the podium question is simply unanswered.

2. **Confidence amplification.** The 9.3 pp advantage in P(top3) for 1-stop shows the aggressive 2-stop is not "hunting the podium" — it is actually *reducing* podium probability in this scenario. This counterintuitive finding is invisible to the `is_top10` model.

3. **The trade-off structure is revealed.** For this context, there is no points-vs-podium trade-off. The 2-stop aggressive strategy is inferior on both dimensions. This changes how a strategy engineer frames the decision.

### Scenario-Conditioned Language

Under our model trained on 2019–2022 data, with the 2023–2024 test set fixed at the SAI Italian Grand Prix 2024 base row, and varying only `n_stops` and `compound_sequence`:

- P(top10 | 1-stop M-H) = **0.956** vs P(top10 | 2-stop S-M-H) = **0.835**
- P(top3 | 1-stop M-H) = **0.559** vs P(top3 | 2-stop S-M-H) = **0.466**

The model does **not** say: "1-stop causes a podium." It says: "Under the training distribution, holding this driver-race context fixed, 1-stop co-occurs with better outcomes on both targets."

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
> "Holding driver pace and team capability fixed at the SAI Italian Grand Prix 2024 base-row values, and varying only strategy inputs, the model predicts these probabilities."

We do **not** claim:
> "Choosing 1-stop causes a podium finish at Monza."

The causal arrow may run from **pace → strategy choice**, not strategy → outcome. This is why the strategy advisor must be combined with telemetry and practice-pace data before race-day deployment.

---

## Conclusion: What This Means for the Strategy Advisor

### For the Strategy Desk

The dual-target analysis for the SAI Monza 2024 scenario reveals:

1. **1-stop M-H is the dominant strategy** for a Tier 1–2 driver starting P5 at a permanent circuit — dominant meaning it is preferred on both targets simultaneously, with no trade-off.

2. **The 2-stop aggressive strategy does not unlock additional podium probability in this context.** This is the counterintuitive finding: the aggressive strategy is not a podium-hunting tool for a competitive car at Monza — it is a risk that reduces both P(top10) and P(top3).

3. **Dual-target modeling adds value even when targets agree.** The confidence amplification across both targets is actionable: it tells the strategy engineer that there is no hidden podium-versus-points trade-off to manage here.

### For Aston Martin Engineering

If the race-day objective is **"maximize constructor points,"** use `is_top10` → 1-stop M-H is the clear recommendation. If the race-day objective is **"take a swing at the podium,"** use `is_top3` → 1-stop M-H is still the recommendation, and the 2-stop does not help. The model removes one decision variable: at Monza for a competitive car, strategy should not be the podium lever — pace is.

This framing is the core value of the Strategy Advisor: it clarifies which decisions the model can inform and which require race-day telemetry and engineering judgment.

---

## Appendix: Reproducibility

To reproduce this analysis, run `hito2_modeling.ipynb` Step 4 with:

```python
# pair_sainz_monza() is defined in Step 4 of hito2_modeling.ipynb
pair = pair_sainz_monza()

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
AGREE: is_top10 prefers '1-stop M-H'; is_top3 prefers '1-stop M-H'.
```

`score_pair()` and `interpret()` are defined in Step 4 of `hito2_modeling.ipynb`. Both functions are required to be in scope before calling. `PRIMARY_TARGET`, `EXPANSION_TARGET`, and `EXPANSION_IS_BINARY` are set in the config cell at the top of the notebook.

**Only `n_stops` and `compound_sequence` are varied.** All other feature values are inherited from the base row identified by `context_filter={"season": 2024, "circuit": "Italian Grand Prix", "Driver": "SAI"}`. This satisfies the Hito 2 leakage policy: scenario inputs only.
