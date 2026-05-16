# Final Report — Capstone Reproducible Report + GitHub Repository

**Module:** Module 5 — Unit IV — Capstone: F1 Race Strategy Advisor  
**Type:** Team deliverable (teams of 2–3)  
**Weight:** 10% of NP (40% of the Capstone grade) — heaviest single Capstone component  
**Deadline:** Sunday, May 17, 2026 — 23:59 CLT  
**Submission:** PDF report uploaded to Canvas + GitHub repo URL with tag `final-v1`

---

## Quick Summary

Submit a reproducible PDF report (8–12 pages) plus a tagged GitHub repository. The report is the integrative narrative of the Capstone — it reframes Hito 1 work, consolidates Hito 2 modeling and error analysis, and adds the Executive Summary and Limitations + Recommendation sections that turn the work from "ML notebook" into "decision-support deliverable for an F1 strategy engineer."

Graded on five dimensions:
- **Sentido de negocio / Domain reasoning (20%)** — does what you built make sense for an F1 strategy engineer?
- **Honestidad del reporte (20%)** — do your claims match your evidence?
- **Technical completeness (25%)** — the floor
- **Reproducibility (20%)** — non-negotiable
- **Written communication (15%)** — the rest

> Submit early Sunday. Use remaining time for Demo Day rehearsal.

---

## What You're Submitting

### 1. The PDF Report

| Field | Requirement |
|---|---|
| Length | 8–12 pages (excl. title page, references, appendix) |
| Format | PDF, single column, 11pt body, 1.0–1.15 line spacing, 1-inch margins |
| Language | English |
| Filename | `IIT414W_FinalReport_<TeamName>.pdf` (no spaces, no special characters) |

- Figures and tables count toward the page budget. Code does not belong in the report.
- Under 8 pages → insufficient depth. Over 12 pages → page-budget deduction.

### 2. The GitHub Repository

- Tagged `final-v1` at the commit matching the submitted PDF. Include the short hash on the title page.
- Runs end-to-end from a fresh clone in **under 10 minutes**, following the README runbook, on Python 3.11 with `environment.yml`.
- `RANDOM_SEED = 414` set in every `random_state=` argument.
- Includes all artifacts from Hito 1 and Hito 2 plus any new files referenced in the Final Report.
- `PROMPTS.md` updated to cover the writing phase.

---

## Title Page Format

```
F1 Race Strategy Advisor:
[Your 1-line finding here]

Team:    [Names]
Course:  IIT414W — Artificial Intelligence Workshop · 2026-1T
Date:    May 17, 2026
Repo:    https://github.com/.../[your-team-repo]
Commit:  [short hash of final-v1 tag]
```

> The 1-line finding is your headline — a sentence a strategy engineer could read in 5 seconds and want to know more.  
> Example: *"Two-stop strategies preserve P(top10) but reduce P(top3) at street circuits."*

---

## Required Sections (9 sections, in order)

| § | Section | Pages | What must appear |
|---|---|---|---|
| 1 | Executive Summary | 1 | Non-technical. Decision supported, headline finding, honest limits, recommendation. Write this last. |
| 2 | Problem Framing | 1–1.5 | Decision context, prediction unit, both targets, scenario variables, ≥3 explicit assumptions with consequences, justified metrics. |
| 3 | Data and Validation | 1–1.5 | Dataset summary, temporal split, leakage audit summary, scenario protocol. |
| 4 | Modeling Approach | 2–2.5 | Baselines, model family per target, calibration approach, feature sets, hyperparameter rationale. Both targets. |
| 5 | Results and Honest Comparison | 2 | Headline metrics table on test set vs docent baseline. ≥1 calibration plot per target. One result per target in plain English. |
| 6 | Error Analysis & What-If | 1.5–2 | Slice analysis on both targets. Three concrete failure-mode hypotheses. One what-if scenario where the two targets disagree. |
| 7 | Limitations and Risks | 0.5–1 | Strategy-confounding limitation explicit. 2–3 other concrete limitations with consequences. Mandatory honesty sentence. |
| 8 | Reproducibility Note & AI Reflection | 0.5 | Pointer to README + PROMPTS.md. One paragraph reflecting on AI use. |
| 9 | References | 0.5 | Course dataset, FastF1, Jolpica, papers, blog posts, course materials. Any consistent style (APA, IEEE, ACM). |

> **Optional appendix** (does not count toward page budget): extended slice tables, additional figures, contribution statement. Do not use the appendix to hide weak analysis.

---

## What's New vs Hito 2

| Section | What's new |
|---|---|
| §1 Executive Summary | New writing. Audience is the team principal, non-technical. |
| §2 Problem Framing | Rewritten from Hito 1 using feedback received Mon May 11. |
| §4 Modeling | Consolidated rationale for design choices across both targets, in prose. |
| §7 Limitations | Extends `mitigations.md` with mandatory honesty sentence + strategy-confounding limitation. |
| §8 AI Reflection | One paragraph of reflection, separate from the raw PROMPTS.md log. |
| §9 References | Properly cited. |

Sections 5 and 6 carry over substantial content from Hito 2 but written as **prose with embedded figures**, not raw markdown checklists.

---

## Submission Policy

- **Deadline:** Sunday May 17, 2026, 23:59 CLT.
- **Late penalty:** −10% of team grade per 24-hour period late.
- **No submission by Tue May 19, 23:59 CLT** → Final Report = 1.0 for the team.
- Demo Day (Mon May 18) happens regardless of report submission status — graded independently.
- Late submissions not eligible for resubmission.

---

## Detailed Rubric

Grade = `0.25 × D1 + 0.20 × D2 + 0.20 × D3 + 0.20 × D4 + 0.15 × D5`  
Scale: Chilean 1.0–7.0, standard 60% bilinear passing-scale conversion.

Performance levels: **Excelente (90–100%)** · **Bueno (70–89%)** · **Suficiente (55–69%)** · **Insuficiente (<55%)**

---

### D1 — Completitud técnica (25%)
*Are all required sections present, complete, and technically correct?*

| Level | Descriptor |
|---|---|
| **Excelente** | All 9 sections with appropriate depth. Both targets in §4–§6. Calibration evidence for all probability outputs. Three concrete failure-mode hypotheses (where / why / how-to-test). What-if scenario shows real disagreement between targets. Metrics table includes docent baseline row. |
| **Bueno** | All 9 sections. Minor gaps: one slice missing, calibration for only one target, or only 2 failure-mode hypotheses. |
| **Suficiente** | Sections present but thin. Second target acknowledged but underanalyzed. Calibration discussed but no plot. |
| **Insuficiente** | Sections missing. Only one target analyzed. No calibration. No what-if. Generic failure modes. |

---

### D2 — Sentido de negocio / Domain reasoning (20%)
*Does what you built make sense for an F1 strategy engineer?*

Four sub-criteria, all required:

1. **Problem clarity** — Decision framed in operational F1 terms: specific user, specific decision, specific time budget.
2. **Solution makes domain sense** — Modeling choices reflect F1 domain reasoning, not generic ML convention.
3. **Concrete value-add** — One specific decision context where the tool helps beyond existing heuristics.
4. **Domain-specific difficulties** — F1-specific and operationally concrete limitations.

| Level | Descriptor |
|---|---|
| **Excelente** | All four sub-criteria fully met. Report would be readable and credible to an actual F1 strategy engineer. |
| **Bueno** | Three sub-criteria fully met; the fourth partially developed. |
| **Suficiente** | Two sub-criteria met. Problem framed mostly in ML terms. Value-add not differentiated from "the model is accurate." |
| **Insuficiente** | One or fewer sub-criteria met. Problem framed entirely in ML terms. No identifiable value-add for a domain user. |

---

### D3 — Honestidad del reporte / comparación justa (20%)
*Do your claims match your evidence?*

| Level | Descriptor |
|---|---|
| **Excelente** | Honest comparison vs docent baseline including underperformance cases. Calibration honestly assessed. §7 contains mandatory honesty sentence with three concrete, testable conditions. What-if framed as observational, not causal. |
| **Bueno** | Mostly honest. Honesty sentence present but one condition vague. One claim in §5 slightly outruns evidence. |
| **Suficiente** | Some claim inflation. Honesty sentence missing or hedged. What-if reads partly as causal claim. |
| **Insuficiente** | Cherry-picked results. No failure discussion. No honesty sentence. Causal language in what-if. |

---

### D4 — Reproducibilidad de terceros (20%)
*Can someone outside your team re-run your work from a fresh clone?*

| Level | Descriptor |
|---|---|
| **Excelente** | Fresh clone runs end-to-end in <10 min. `RANDOM_SEED = 414` everywhere. Paths work from fresh checkout. All dependencies pinned. `PROMPTS.md` current. Tag `final-v1` matches title page hash. Notebook outputs match report figures. |
| **Bueno** | Reproducible with one or two minor manual fixes. Seeds set in most places. README mostly clear. |
| **Suficiente** | Reproducible but requires significant manual setup. Some figures cannot be regenerated. |
| **Insuficiente** | Cannot be reproduced from a fresh clone. Missing seeds, broken runbook, hardcoded paths, or commit hash mismatch. |

---

### D5 — Comunicación escrita (15%)
*Is the writing clear? Are figures publication-quality? Is the page budget respected?*

| Level | Descriptor |
|---|---|
| **Excelente** | Publication-quality figures (clear labels, purposeful color, consistent style). Page budget respected (8–12 pages). English clear and precise. Every number ties to a decision or interpretation. |
| **Bueno** | Generally clear writing. Minor formatting issues or one weak figure. Page budget respected. |
| **Suficiente** | Writing readable but uneven. Figures functional but unclear. One section over/under page budget. |
| **Insuficiente** | Hard to read. Over 12 or under 8 pages. Figures unlabeled. Section order inconsistent. |

---

## What Good Looks Like vs Common Pitfalls

### A strong report:
- Sounds like F1, not like a textbook.
- Articulates a concrete value-add vs existing heuristics.
- Surfaces a non-obvious trade-off between the two targets in §6.
- Includes an honesty sentence in §7 with three conditions testable by a peer.
- Shows calibration evidence — not just discrimination metrics — for every probability output.
- Translates between technical (§3–§6) and non-technical language (§1, §7) without losing precision.

### A weak report:
- Reads as an ML lab report with F1 wallpaper.
- Hides behind metrics without telling a decision story.
- Treats the two targets as separate exercises.
- States generic ML caveats instead of F1-specific risks.
- Has a `PROMPTS.md` that doesn't match §8.
- Exceeds page budget by padding the methodology.
- Submits figures that don't match notebook output.

---

## Connection to Demo Day (Mon May 18)

| | Final Report | Demo Day |
|---|---|---|
| Audience | Reader with 20 min | Jury with 7 min + 5 min Q&A |
| Format | PDF, written | Pitch, live |

- §6 what-if comparison → your Demo Day slide 4.
- §7 honesty sentence → your Demo Day slide 5. Memorize it.
- Q&A is individual. Every team member must be able to defend every section.

---

## AI Use Policy

`PROMPTS.md` must cover the writing phase. Document AI use for:
- Drafting or restructuring prose sections
- Generating figure captions or table summaries
- Critiquing your own writing
- Translating technical content into Executive Summary language

At least **one rejected or corrected AI suggestion** must be documented in `PROMPTS.md`.  
Use the 6-field standard: **Context · Prompts · Output · Validation · Adaptations · Final Decision**

§8 contains one paragraph of reflection — do not duplicate the raw log there, just point to it.

---

## Submission Checklist

### Document and structure
- [ ] PDF length is 8–12 pages (excl. title page, references, appendix)
- [ ] PDF filename: `IIT414W_FinalReport_<TeamName>.pdf`
- [ ] All 9 required sections present and in prescribed order
- [ ] English, 11pt body, 1-inch margins

### Technical content (D1)
- [ ] Both targets discussed in §4, §5, and §6
- [ ] At least one what-if comparison where the two targets disagree (§6)
- [ ] Calibration evidence for every probability output discussed
- [ ] Three concrete failure-mode hypotheses in §6 (where / why / how-to-test)

### Domain reasoning (D2)
- [ ] §2 names a specific user, a specific decision, and a specific time budget
- [ ] Scenario inputs framed as what-if controls in §3 and §4; metric choices justified against the strategy decision
- [ ] §6 names one specific decision context where the tool helps beyond existing heuristics
- [ ] §7 names F1-specific limitations (car pace confounding, untestable counterfactuals, regime shifts, etc.)

### Honesty (D3)
- [ ] Honest comparison vs docent baseline in §5, including underperformance cases
- [ ] Honesty sentence in §7 with three concrete conditions
- [ ] Strategy-confounding limitation addressed explicitly

### Reproducibility (D4)
- [ ] GitHub repo tagged `final-v1` with matching commit hash on title page
- [ ] README runbook reproduces report figures end-to-end in <10 min from a fresh clone
- [ ] `PROMPTS.md` updated to include writing phase, with ≥1 rejected/corrected AI suggestion

### Submission
- [ ] PDF uploaded to Canvas assignment
- [ ] GitHub URL pasted in Canvas submission text field
