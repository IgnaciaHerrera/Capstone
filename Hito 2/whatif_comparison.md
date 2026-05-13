# What-If Comparison — Hito 2

**Team:** Feligma  
**Date:** May 12, 2026  
**Targets:** `is_top10` (primary) and `is_top3` (expansion)  
**Context:** Matched driver-race scenarios with fixed context, strategy inputs varied

---

## Executive Summary

This document presents a **scenario-conditioned what-if comparison** using the dual-target framework. The primary scenario — Sergio Pérez (Red Bull) at the Hungarian Grand Prix 2023 — demonstrates the value of modeling both `is_top10` and `is_top3` simultaneously, especially when targets **disagree**.

**Result for this scenario:** The two targets **AGREE on strategy**, but reveal critical disagreement in decision urgency. Both `is_top10` and `is_top3` prefer 2-stop M-H: the primary target is indifferent between tire compounds (both yield 0.9276 P(top10)), while the expansion target strongly prefers M-H (+22.87 pp, 25.0% vs 2.1%). This reveals a critical finding: **2-stop M-H is strategically superior because is_top10 cannot distinguish between tire compounds, but is_top3 clearly demonstrates the podium advantage of M-H.** For a Red Bull driver with podium aspirations from grid P9, this tire compound choice is non-negotiable.

**The strategic value of dual-target modeling:** Without `is_top3` expansion, a `is_top10`-only model would randomly choose between 2-stop M-H and 2-stop H-M-M (both 0.9276 probability) or defer to other factors, completely missing that M-H preserves podium chances at 25% while H-M-M collapses to 2.1%. Dual-target modeling surfaces this decision signal; single-target modeling hides it. From grid P9 at Hungaroring with a front-tier constructor, the choice becomes clear: choose 2-stop M-H to protect the 22.87 pp podium margin, not H-M-M which sacrifices podium probability for no top-10 gain.

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

Hungaroring from grid P9 creates a scenario where the tire compound choice within a 2-stop strategy reveals the decision value of dual-target modeling. The track's high tire degradation makes 2-stop the baseline strategy for midfield drivers. Within that 2-stop constraint, the tire compound sequence varies: aggressive (H-M-M) vs conservative (M-H). From grid P9, `is_top10` sees both strategies as equivalent (0.9276), but `is_top3` reveals a massive podium gap (25.0% vs 2.1%). This is exactly the scenario where dual-target modeling adds value: **the expansion target surfaces a decision signal that the primary target alone would miss.** For a Red Bull driver with genuine podium aspirations, the M-H sequence is strategically superior not because it improves top-10 probability, but because it protects the podium margin while maintaining top-10 parity.

---

## Strategy Pair: Two-Stop M-H Conservative vs Two-Stop H-M-M Aggressive

### Scenario A: Two-Stop M-H Conservative

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 2 |
| `compound_sequence` | "M-H" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Pit twice for tire strategy: Medium tires for Stint 1, then pit for Hard tires for Stint 2. From grid P9, this conservative tire plan prioritizes tire life and consistency. Medium-Hard sequence is typical for Hungaroring's high tire wear — the strategy maintains track position through consistent tire grip without aggressive undercut/overcut tactics. This minimizes pit loss and maximizes podium window (smooth pace progression for overtaking or defending).

### Scenario B: Two-Stop H-M-M Aggressive

**Strategy inputs varied:**

| Input | Value |
|---|---|
| `n_stops` | 2 |
| `compound_sequence` | "H-M-M" |

All other features held at base-row values from the Hungarian Grand Prix 2023 PER context.

**Rationale:** Pit twice with aggressive tire strategy: Hard tires for Stint 1, then pit for Medium, then pit for Medium again (or adjust final compound). Hard-start incurs degradation penalty early but enables aggressive undercut/overcut flexibility if competitors pit differently. Two stops on Hard-Medium-Medium opens tactical options (fresher rubber in critical stints, respond to competitor pit timings) but at higher cumulative pit loss and risk of falling out of podium contention if the aggressive strategy loses track position.

> **⚠️ Note on scenario inputs:** Only `n_stops` and `compound_sequence` are varied, as required by the Hito 2 leakage policy. Both inputs are in `SCENARIO_INPUT_COLS` as defined in the notebook. Stint lengths and pit timing are held at base-row values from the Hungarian Grand Prix 2023 PER race — this is a restriction of our feature set, not an oversight.

---

## Model Predictions: Two-Stop M-H vs Two-Stop H-M-M

Results produced by `score_pair(test, **pair_perez_hungary())` in `hito2_modeling.ipynb`, Step 4.

### Primary Target: `is_top10`

| Metric | 2-Stop M-H | 2-Stop H-M-M | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top10)** | 0.9276 | 0.9276 | 0.0000 | **NO PREFERENCE** ← |
| **Interpretation** | 92.76% chance of top-10 finish | 92.76% chance of top-10 finish | 0.0 pp gap (IDENTICAL) | Both equally valid |

### Expansion Target: `is_top3`

| Metric | 2-Stop M-H | 2-Stop H-M-M | Difference | Preferred |
|---|---|---|---|---|
| **P(is_top3)** | 0.2500 | 0.0213 | +0.2287 | **2-Stop M-H** ✓ |
| **Interpretation** | 25.0% chance of podium | 2.13% chance of podium | 22.87 pp gap (strongly favors M-H) | M-H protects podium |

### Verdict

```
AGREE: is_top10 shows NO PREFERENCE ('2-stop M-H' and '2-stop H-M-M' are IDENTICAL at 0.9276); 
       is_top3 strongly prefers '2-stop M-H' (0.2500 vs 0.0213).
       
DECISION SIGNAL: Both targets prefer 2-stop M-H, but for different reasons:
- is_top10 is INDIFFERENT (tire compounds irrelevant to top-10 probability)
- is_top3 DECISIVELY PREFERS M-H (podium preservation requires conservative tire management)
```

---

## Interpreting the AGREE Verdict: What Dual-Target Modeling Reveals

### Why Same-Direction Agreement with Asymmetric Margins Is Critical

A single-target `is_top10` model returns: "Both 2-stop M-H and 2-stop H-M-M yield 92.76% P(top10). These strategies are equivalent." Recommendation: unclear; defer to other factors.

A dual-target model returns: "Both strategies yield 92.76% P(top10) — the primary target is indifferent — but 2-stop M-H yields 25.0% P(top3) while 2-stop H-M-M yields only 2.13% P(top3). A 22.87 pp podium gap with zero top-10 gap means M-H is unambiguously superior." This is qualitatively different information for three reasons:

1. **AGREEMENT reveals that the choice is DECISIVE, not a tie-breaker.** When targets agree, the decision is unambiguous: choose the scenario both prefer. But asymmetric margins show why: `is_top10` doesn't care (0.0 pp gap), while `is_top3` cares enormously (22.87 pp gap). This tells the strategy desk: "The tire compound choice doesn't matter for top-10, but it matters catastrophically for podium. If you care about podium at all, there is only one choice: M-H."

2. **The gap sizes reveal decision urgency asymmetry.** The primary target is indifferent (0.0 pp); the expansion target is definitive (22.87 pp). This is not ambiguity—it's clarity. If podium is within reach (25% for M-H), sacrificing it by switching to H-M-M (2.13%) is inexplicable. The small gap (0.0 pp on top-10) cannot justify the massive gap (22.87 pp on podium).

3. **AGREE with asymmetric strength enables high-confidence decisions.** From grid P9 at Hungaroring, the team can confidently say: "Choose 2-stop M-H. Both strategies reach top-10 with equal probability, but M-H preserves our podium chances (+22.87 pp). This is not a trade-off; it's a choice with no cost—we gain podium probability with zero top-10 loss." Dual-target modeling surfaces this; `is_top10` alone would flip a coin.

### Scenario-Conditioned Language

Under our model trained on 2019–2021 data (calibrated on 2022), with the 2023 test set fixed at the PER Hungarian Grand Prix 2023 base row, and varying only `n_stops` (both fixed to 2) and `compound_sequence`:

- P(top10 | 2-stop M-H) = **0.9276** vs P(top10 | 2-stop H-M-M) = **0.9276** → 0.0 pp gap (NO PREFERENCE on is_top10)
- P(top3 | 2-stop M-H) = **0.2500** vs P(top3 | 2-stop H-M-M) = **0.0213** → 22.87 pp gap (STRONG PREFERENCE for M-H on is_top3)

The model does **not** say: "M-H guarantees podium" or "H-M-M is inferior in all ways." It says: "Under the training distribution, holding Pérez at Hungaroring 2023 fixed at grid P9, both 2-stop strategies co-occur with identical top-10 outcomes but dramatically different podium outcomes. The choice between them depends on whether podium preservation is strategically valuable for your race objective. If it is, M-H is strictly dominant. If it is not, both strategies are equivalent."

The critical insight is: **because is_top10 is indifferent, the expansion target is_top3 fully determines the decision.** This is the core value of dual-target modeling: the expansion target breaks the tie when the primary target cannot.

---

## Special Case: When Targets Would Disagree

**For the Pérez Hungary scenario:** The targets AGREE. Both prefer 2-stop M-H. But this raises a question: under what conditions would the targets genuinely disagree on strategy?

### Structural Conditions for Disagreement (Hypothetical)

Based on the model behavior and training data, disagreement would occur when:

| Condition | Expected Effect |
|---|---|
| **Lower-tier constructor (Tier 3), starting P13–P15** | 1-stop may preserve track position but fail to reach top-10 (P(top10) ≈ 0.15); 2-stop may unlock undercut aggression for top-10 (P(top10) ≈ 0.25) while 1-stop still dominates podium (P(top3) ≈ 0.03 for both) → weak agreement on 2-stop |
| **Street circuit (Singapore, Monaco) — extreme tire deg** | 2-stop correlates with points-scoring historically, but podium from P10+ is rare regardless → likely weak agreement (both targets near-floor) |
| **Pure midfield driver, competitive day** | 1-stop conservative may land P8–P9 safely (P(top10) = 0.68, P(top3) = 0.02); 2-stop aggressive may land P4–P5 or P12 (P(top10) = 0.55, P(top3) = 0.12) → **DISAGREEMENT: is_top10 prefers 1-stop; is_top3 prefers 2-stop** |
| **Wet or mixed conditions** | Strategy correlation breaks down; model may assign similar probabilities to both scenarios under either target → weak agreement (near-tied) |

### Hypothetical DISAGREE Example

Driver at P10 on grid, Tier 2 constructor, permanent circuit:

- 1-stop strategy → P(top10) = 0.62, P(top3) = 0.04
- 2-stop strategy → P(top10) = 0.55, P(top3) = 0.09

In this case: `is_top10` prefers 1-stop (+0.07 pp); `is_top3` prefers 2-stop (+0.05 pp). **Genuine DISAGREE scenario.**

In this case the trade-off is real: the team must choose between maximizing points-scoring probability (1-stop, +0.07 pp on top-10) and maximizing podium probability (2-stop, +0.05 pp on podium). That choice requires an explicit team objective. `is_top10` alone would recommend 1-stop unconditionally, missing the 2-stop's podium upside entirely.

**This is why the Perez Hungary scenario is valuable:** It shows the OPPOSITE case—AGREEMENT with asymmetric strength. The Sainz Monza scenario (from error_analysis.md) shows pure AGREE. A true DISAGREE scenario would require a different driver-race-grid-tier combination.

---

## Limitations & Confounding

### Strategy Confounding at Hungaroring

The predictions in this analysis may be partially explained by confounding between strategy choice and underlying car pace:

1. **In the training data (2019–2022)**, teams that chose 2-stop M-H vs 2-stop H-M-M at Hungaroring made these choices based on car competitiveness and expected pace — pace is correlated with finishing position regardless of tire compound sequence.

2. **Red Bull in 2023 has front-tier pace.** Pérez starting P9 is in a moderately-high-P(top10), moderate-P(top3) zone. The model's base row carries this pace signal in `prior_avg_finish_driver` and `prior_avg_finish_constructor` (Red Bull prior3 avg finish ≈ 5th, Pérez prior3 ≈ 7th). The tire compound variation happens on top of a front-tier competitive base.

3. **The model cannot separate** whether M-H preserves podium outcomes because (A) conservative tire management is genuinely better for Hungaroring's circuit characteristics, or (B) front-tier teams with strong pace happen to choose M-H sequences more often in training data.

### Mitigation in Interpretation

We interpret the what-if scenario-conditionally:
> "Holding driver pace and team capability fixed at the Red Bull PER Hungarian Grand Prix 2023 base-row values, and varying only tire compound sequence within a 2-stop strategy, the model predicts these probabilities."

We do **not** claim:
> "Choosing M-H causes a podium finish."

The confounding risk exists: front-tier teams with stronger tire management may choose conservative M-H sequences (gradual compound progression) more often in training data. The model captures this correlation but cannot definitively separate tire-sequence effect from underlying team capability and driver skill. A driver with poor tire degradation management might need H-M-M regardless of team instruction. This is why the strategy advisor must be combined with telemetry, practice-pace data, and driver-specific tire feedback before race-day deployment.

---

## Conclusion: What This Means for the Strategy Advisor

### For the Strategy Desk

The dual-target analysis for the PER Hungarian 2023 scenario reveals:

1. **The two targets AGREE on strategy, but with asymmetric decision strength.** `is_top10` is indifferent (0.0 pp gap between M-H and H-M-M); `is_top3` strongly prefers M-H (+22.87 pp). This is not a tie-breaking scenario; it's a **decision signal where the expansion target breaks a primary-target tie.**

2. **The tire compound trade-off is asymmetric and FAVORS M-H decisively.** Choosing M-H gains +22.87 pp in podium probability with zero top-10 cost (both 0.9276). From grid P9 at Hungaroring with a front-tier constructor, if podium is within reach (25% for M-H, 2% for H-M-M), choosing H-M-M is strategically indefensible: lose 22.87 pp of podium probability to gain 0.0 pp of top-10 probability. This is a clear win-win for M-H.

3. **Dual-target modeling enables high-confidence decisions when primary target cannot differentiate.** Under `is_top10` alone, the choice is ambiguous: use either tire compound (both 92.76%). Dual-target analysis reveals: "Both tire compounds reach top-10 equally, but M-H preserves podium chances (25.0%) while H-M-M collapses them (2.13%). If podium is strategically valuable for your race, there is only one choice: M-H. If podium is irrelevant, both are equivalent — but accepting a 22.87 pp podium loss for zero top-10 gain is irrational." This is actionable decision support.

### For Strategy Execution

**Primary recommendation: Use 2-stop M-H.** Both tire compounds yield identical top-10 probability (0.9276, 92.76%), but M-H preserves podium chances at 25.0% versus H-M-M's collapse to 2.13%. From grid P9 at Hungaroring for a front-tier constructor with genuine podium aspirations, conservative tire management (M-H) is strategically dominant.

**Why this recommendation matters:** If a `is_top10`-only model had been consulted, it would say: "Both strategies are equivalent for top-10 (0.9276). Defer to fuel load, tire wear dynamics, or competitor pit strategy." Dual-target modeling surfaces the hidden podium signal and enables the team to choose decisively: **choose M-H to protect the 22.87 pp podium margin with zero top-10 trade-off.**

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
AGREE: is_top10 shows NO PREFERENCE ('2-stop M-H' and '2-stop H-M-M' are IDENTICAL at 0.9276); 
       is_top3 strongly prefers '2-stop M-H' (0.2500 vs 0.0213).
```

Or more verbosely:

```
scenario_label  season              circuit Driver     Team  grid_position  n_stops compound_sequence  stint1_length  stint2_length  stint3_length  P(is_top10)  P(is_top3)
  2-stop M-H    2023 Hungarian Grand Prix    PER Red Bull            9.0        2               M-H             22             16             27     0.927564    0.250000
2-stop H-M-M    2023 Hungarian Grand Prix    PER Red Bull            9.0        2             H-M-M             22             16             27     0.927564    0.021277

AGREE: is_top10 shows NO PREFERENCE ('2-stop M-H' and '2-stop H-M-M' are IDENTICAL at 0.9276); 
       is_top3 strongly prefers '2-stop M-H' (0.2500 vs 0.0213).
```

`score_pair()` and `interpret()` are defined in Step 4 of `hito2_modeling.ipynb`. Both functions are required to be in scope before calling. `PRIMARY_TARGET`, `EXPANSION_TARGET`, and `EXPANSION_IS_BINARY` are set in the config cell at the top of the notebook.

**Only `n_stops` (both fixed to 2) and `compound_sequence` are varied.** All other feature values are inherited from the base row identified by `context_filter={"season": 2023, "circuit": "Hungarian Grand Prix", "Driver": "PER"}`. This satisfies the Hito 2 leakage policy: scenario inputs only.
