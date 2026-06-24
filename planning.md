# TakeMeter Project Planning

## Community

My project uses r/soccer, one of the largest online soccer communities with more than 8 million members. The subreddit includes discussions about matches, players, and clubs from around the world, along with transfer rumors and ongoing debates about player performance, tactics, and reactions to global leagues and tournaments. Users contribute a mix of analysis, hot_take, and reaction, ranging from detailed breakdowns to immediate emotional responses and unsupported opinions. This makes the community well-suited for classification, since users frequently call out “bad” takes while also praising strong analysis when it appears.

## Label Taxonomy

I am using 3 labels: **analysis**, **hot_take**, and **reaction**.

---

### Label 1: analysis

**Definition:**
The post supports a football claim using tactical reasoning, statistics, historical comparison, or detailed football knowledge.

**Example 1 (from r/soccer):**
"Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions (2006, 2010, 2014, 2018, 2022 & 2026)."
→ Provides specific, verifiable historical data to support a claim about Ronaldo's career.

**Example 2 (from r/soccer):**
"Curaçao's Eloy Room recorded 15 saves against Ecuador, the most on record (since 1966) by any goalkeeper in a FIFA World Cup match that did not feature extra-time."
→ Uses a verifiable statistic with historical context to make a meaningful point.

**Uncertain Example:**
"Spain's Head Coach Luis de la Fuente: 'Rodrigo is the best player in the world, and even at 50% he's much better than most midfielders in the world.'"
→ Could be analysis (expert opinion from a coach) or hot_take (bold claim without supporting evidence).

**Decision Rule:**
If the post provides specific, verifiable evidence or reasoning that would support the claim even without the opinion framing, label it analysis. If the claim is primarily bold and the evidence is thin or missing, label it hot_take.

---

### Label 2: hot_take

**Definition:**
A strong, confident football opinion stated with little or no supporting evidence — the post asserts rather than argues.

**Example 1:**
"Palmer is already world class."
→ Bold claim, no evidence or reasoning provided.

**Example 2:**
"Real Madrid would win the Premier League every season."
→ Confident opinion with no supporting argument.

**Uncertain Example:**
"Haaland isn't as complete as Kane because he only contributes goals."
→ Gives a reason ("only contributes goals") but no actual evidence or deeper argument.

**Decision Rule:**
If removing the opinion framing leaves behind a meaningful football argument with real evidence, label it analysis. If all that remains is the bold claim, label it hot_take.

---

### Label 3: reaction

**Definition:**
An emotional response to a football event — a goal, a loss, a transfer, a referee decision — where the primary purpose is expressing feeling rather than making an argument.

**Example 1 (from r/soccer):**
"WHAT A GOAL!!!"
→ Pure emotional response to a goal clip.

**Example 2 (from r/soccer):**
"Cape Verde goalkeeper Vozinha after the draw vs Spain: 'I worked all my life for this. For this moment, for this dream.'"
→ Emotional response to a match result, no analytical content.

**Uncertain Example:**
"I can't believe the manager subbed him off."
→ Could be reaction (emotional frustration) or the start of a hot_take if the post goes on to make a claim.

**Decision Rule:**
If the primary purpose is expressing a feeling in response to a specific event, label it reaction. If the post pivots into making a claim or argument, re-evaluate as hot_take or analysis.

---

## Hardest Edge Case

**Example:**
"Thierry Henry in his post-match analysis about Ronaldo after Portugal's game vs DR Congo: 'The team needs to score, not you need to score.'"

This could be:
- **analysis** — Henry is an expert making a tactical/strategic point about team vs. individual goals
- **hot_take** — it's a pundit quote with no data behind it, just a strong opinion

**Decision Rule:**
Ask: "If I remove the speaker's name and credentials, is there still a meaningful argument here?"
If yes → analysis. If it only works because of who said it → hot_take.
The Henry quote → **hot_take** (the point is asserted, not argued with evidence).

---

## Why These Labels Matter

The community of r/soccer actively processes and categorizes posts based on their level of argumentation and reasoning rather than their emotional value. In evaluating quality discourse, members of the r/soccer community often refer back to posts where someone presents an opinion with confidence or a strong reaction. In comments on these posts, members will often call someone out for a “hot take” or offer praise for “real analysis.”

---

## Data Collection

- **Source:** r/soccer (post titles from the front page, match thread comments, daily discussion comments)
- **Collected from:** r/soccer front page, France vs Iraq match thread, Norway vs Senegal match thread, Daily Discussion thread
- **Total examples:** 261
- **Label distribution:** analysis: 68 | hot_take: 95 | reaction: 98
- **Split:** 70% train / 15% validation / 15% test
- **Format:** CSV with columns: `id`, `text`, `label`

---

## 3 Hard-to-Label Examples

**1. "Thierry Henry in his post-match analysis about Ronaldo after Portugal's game vs DR Congo: 'The team needs to score, not you need to score.'"**
Could be analysis (expert making a tactical point) or hot_take (pundit opinion with no evidence). Labeled **hot_take** — if you remove Henry's name and credentials, the claim has no supporting argument behind it.

**2. "Iraq made so many fatal errors. 1 error led to goal 1, another error led to goal 2."**
Could be analysis (breaking down what happened tactically) or hot_take (surface-level observation). Labeled **hot_take** — the reasoning is too vague to count as real analysis; it describes what happened without any meaningful breakdown.

**3. "Anyway, his situation is morbidly funny. 3 UCLs between his 2 clubs in 3 years and him not winning any."**
Could be reaction (expressing amusement) or analysis (using a stat to make a point). Labeled **analysis** — the specific stat (3 UCLs, 3 years, 0 wins) is the actual substance of the point, not just color for a feeling.

---

## Evaluation Metrics

I'll use the following metrics to evaluate both the fine-tuned DistilBERT model and the Groq zero-shot baseline:

- **Accuracy** — the percentage of accurate predictions made across the three labels overall. The interpretation of accuracy would provide some basic information, but alone it cannot provide a detailed view with respect to label(s) where predictions may be failing as the model is predicting differently for different labels.
- **F1 score per class** — a measure of the harmonic mean between precision of predictions received for a given label and the recall rate on that same label. The F1 score is essential due to the fact that my dataset is not perfectly balanced, and I would like to know if the model is consistently failing for one of the three labels (for example; missing "hot_takes" but modeling well for "reactions").
- **Confusion matrix** — provides insights into exactly which labels have been confused with other labels. I expect the largest boundary confusion will occur between “hot_take” vs. “analysis”, therefore my emphasis is on examining how many times these labels have been confused with one another.

A model that predicts “reaction” across the board would ultimately receive an accuracy of ~38% when referencing actual distribution of predictions/responses across voting categories. This cannot serve as an effective model. The per-class F1 score will allow me to evaluate whether or not the model is providing me with any level of distinction across my labels.

---

## Definition of Success

For this classifier to be genuinely useful in a real community tool, I'd set these thresholds:

- **Overall accuracy ≥ 70%** on the test set for the fine-tuned model
- **F1 ≥ 0.60 for each individual label** — no single label should be completely missed
- **Fine-tuned model outperforms the Groq zero-shot baseline** by at least 5 percentage points in overall accuracy — otherwise fine-tuning didn't add value

"Good enough for deployment" means the model correctly classifies at least 7 out of 10 posts, and doesn't systematically fail on any one label. If the model hits 85%+ accuracy, I'll check for test set leakage or labels that are too easy.

---

## AI Tool Plan

**Label stress-testing:**
I used Claude to create boundary posts for hot takes versus analysis; this turned out to be the hardest pair in my taxonomy. For example, “Due to multiple major errors in Iraq, one error led to goal 1 and then a second error led to goal 2.” This process also highlighted posts that didn’t clearly fit either category and pushed me to refine the decision rules. After reflecting on it, a valid analysis needs some form of verifiable evidence, rather than just describing the event.

**Annotation assistance:**
I used Claude to pre-label a full dataset of 261 examples using my labeling rules and decision criteria. I then checked the distribution and spot-checked around ~30 rows for additional verification. All pre-labeled examples are included in this document, so the dataset is AI-assisted with human review.

**Failure analysis:**
Once I run the model, I’ll give Claude the list of incorrect predictions and ask it to identify any patterns (e.g., whether it consistently confuses short hot_takes with short reaction posts, or struggles with quotes from experts). After identifying any patterns, I will manually review those cases to confirm them before writing up my report.
