# Pair Review Worksheet — Capstone Hito 1
### IIT414W · Block 5 · Mon May 4, 2026 · 15:45–16:05

> **The point of this exchange is structured scrutiny, not feedback.** Polite reviews are useless. Your job for the next 14 minutes is to find the weakest decision in the partner team's framing and name it concretely. The team being reviewed commits the critique they received to GitHub at 16:00 — public artifact, no escape.

**Reviewing team:** ____________________
**Reviewed team:** ____________________

---

## How this works (instructor reads aloud at 15:45)

1. **Minutes 1–7:** read the partner team's `framing.md` and look at their dataset-load notebook. Write your answers below to at least 3 of the 5 questions.
2. **Minutes 8–14:** structured conversation. Each team gives the other ONE concrete critique — the strongest one you found. Not three. Not five. ONE that lands.
3. **Minutes 15–18:** each team writes the critique they RECEIVED into their own `framing.md` under section 8 ("Critique Received").
4. **Minute 19:** instructor calls time. Critique-received sections committed to GitHub.

There is no debate phase. The reviewed team writes down the critique, decides whether they agree, and writes a 1-line plan for how they'll address it. The reviewer's job is to deliver the critique, not to defend it.

---

## The Five Required Review Questions

For each question, write a concrete answer based on what you read in the partner team's framing.md. "Looks good" is not a concrete answer. You must give answers to at least 3 of the 5.

### Q1. Does their target match their decision context, or is `is_top10` chosen because it's the easiest binary?

> *Look at their Section 1 (decision context) and Section 2 (target). If their decision is about podiums, is_top10 is too coarse. If their decision is about points generally, is_top10 might be reasonable. Is_top10 is locked for Hito 1, but their framing should still acknowledge if a different target would fit better.*

Concrete answer:

> 

---

### Q2. Is their baseline plan F1-defendable? Could they justify it WITHOUT ever seeing 2023–2024 data?

> *Look at their Section 3. Did they describe the baseline based on F1 logic (qualifying → grid → constructor tier → recent form), or did they describe it based on what they think will work on the test set? The first is defendable. The second is contaminated reasoning.*

Concrete answer:

> 

---

### Q3. Are their what-if scenarios specific enough to RUN, or are they generic?

> *Look at their Section 4. Do their scenarios have actual feature values (e.g. "n_stops=1, compound_sequence=M-H, stint_lengths=[35, 35]")? Or do they say something vague like "we'll compare 1-stop vs 2-stop strategies"? Generic scenarios cannot be executed against the model on Wednesday.*

Concrete answer:

> 

---

### Q4. Which of the five known limitations did they NOT acknowledge that they should have?

> *The five disclosed limitations: (1) coverage starts 2019, (2) qualifying_position is a stand-in, (3) safety_car is binary not interval-counted, (4) strategy features are post-race observations, (5) strategy choice is confounded with car/driver/weather. Look at their Section 5. Which one bites their specific approach but they didn't write down?*

Concrete answer:

> 

---

### Q5. If their model lands at Brier 0.20 on the test set (worse than docent grid-rule 0.208 — close to it but not better), what does their framing currently say to defend that scenario?

> *This is the "what if my model isn't good enough" question. Look at their Section 6 (experiment plan). If their framing doesn't currently have a path for "we did not beat the docent baseline, here's what we'd say," they will be exposed in Demo Day. The strongest framings have a fallback story.*

Concrete answer:

> 

---

## The ONE Concrete Critique We Will Deliver

After answering 3+ questions above, decide: which critique is the most important for this team to hear? Write it as one sentence, framed as an observation, not an attack.

**Format:** "Your [section X] doesn't [specific issue]. The consequence is [what happens in Hito 1 or Demo Day]. One thing to do: [concrete action]."

> 

**Example of a strong critique:** "Your Section 4 lists 'compare 1-stop vs 2-stop' but doesn't specify driver, circuit, or compound. The consequence is your Hito 1 won't have an executable what-if — Wednesday's TA can't help with that. One thing to do: pick three rows from the dataset (one driver, one circuit, three n_stops values) and write the specific scenarios in Section 4 before 15:40."

**Example of a weak critique:** "Your framing is good but could be more specific in Section 4."

---

## Notes for Reviewing Team (your records, not committed)

What did you learn from reading their framing that informs your own?

> 

What is one thing they did better than you did?

> 

---

## After the Exchange

The reviewed team writes the critique they received into their own `framing.md` Section 8 by 16:00. The reviewing team's worksheet is for their own records — keep it as a learning artifact.

Instructor records pair review participation in the session log. Pairs that visibly went through the questions vs pairs that just chatted will be visible from the artifact (the critique-received section).
