# Team Decision Sheet — Capstone Hito 1
### IIT414W · F1 Race Strategy Advisor · Mon May 4, 2026

> **Instructions.** Complete this sheet in your team repo as `framing.md`. Every team has 60 minutes during the studio block (14:45–15:45). Required commits: by 15:00 (sections 1–4 populated) and by 15:40 (full sheet + dataset-load notebook). No section can be left blank — write "TBD with rationale" if you are uncertain, but blank entries fail the framing rubric.

**Team name:** Feligma
**Team members:** Martín Guerrero, Felipe Vázquez e Ignacia Herrera
**GitHub repo URL:** https://github.com/IgnaciaHerrera/Capstone.git

---

## 1. Decision Context

**What strategy decision is this tool supporting?**
> One sentence. Example: "Whether to recommend a 1-stop vs 2-stop strategy for a midfield-grid driver at a circuit with predictable degradation."

**Who makes this decision?**
> One sentence. Example: "The strategy desk on the pit wall during a Friday strategy meeting."

**When in the race weekend is the decision made?**
> One sentence. Example: "Friday evening, after FP2, before parc fermé conditions lock the car setup."

---

## 2. Target & Metric

**Target (LOCKED for Hito 1):** `is_top10`

**Primary metric:** ____________________

**Why this metric for this decision?** (2 sentences max — what does the metric measure that an alternative does not?)

> 

**Secondary metric (optional but recommended):** ____________________

**Temporal split (LOCKED for Hito 1):**
- Train: seasons 2019, 2020, 2021
- Calibration: season 2022 (used to fit calibration mapping; never for model selection)
- Test: seasons 2023, 2024 (untouched until final evaluation)

---

## 3. Baseline Plan

**Baseline approach (one sentence):**
> Example: "Calibrated logistic regression on grid_position + constructor_tier + n_stops only."

**Why is this baseline F1-defendable?** (One sentence — could you justify it without ever seeing 2023–2024 data?)

> 

**Direction check:** higher baseline score means higher predicted P(top10). Yes / No / Explain.

> 

**Expected baseline performance vs docent floor:**
- Grid-rule docent baseline: Brier = 0.208 on test
- Calibrated docent model: Brier = 0.132 on test, ROC-AUC = 0.892
- Our team's best baseline expected to land near: Brier = ____________________

---

## 4. What-If Comparison Plan

**Strategy variables we will vary:**
- [ ] `n_stops`
- [ ] `compound_sequence`
- [ ] `stint_lengths` (or stint1_length, stint2_length, etc.)
- [ ] `avg_pit_stop_duration_s`
- [ ] Other: ____________________

**Concrete scenarios to compare (at least two, with specific values):**

> Scenario A: ____________________
> Scenario B: ____________________

**Decision metric for the comparison:**
> Example: "Difference in calibrated P(is_top10) between Scenario A and Scenario B, with bootstrap 90% confidence interval."

---

## 5. Limitations Acknowledgment

**Five known dataset limitations are documented in the Capstone Brief. Which TWO most affect our team's specific approach?**

Limitation #1 we acknowledge: ____________________

> Why it matters for our approach (1 sentence):

Limitation #2 we acknowledge: ____________________

> Why it matters for our approach (1 sentence):

---

## 6. Experiment Plan for Hito 1

**Three experiments we will run between today and Wednesday 16:20:**

1. ____________________
2. ____________________
3. ____________________

**Hypothesis for each (one line each — what do we expect to happen and why?):**

> 1. 
> 2. 
> 3. 

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
