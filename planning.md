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

Members of r/soccer actively distinguish between posts that make real football arguments versus those that just state opinions confidently versus pure emotional reactions to events. These three categories reflect how the community itself evaluates discourse quality — calling out "this is just a hot take" or praising "actually good analysis" is common in comment sections.

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

- **Accuracy** — overall percentage of correct predictions across all three labels. Useful as a quick summary but not enough on its own because it can hide per-class failures.
- **Per-class F1 score** — the harmonic mean of precision and recall for each label individually. This matters because my dataset isn't perfectly balanced, and I want to know if the model is systematically failing on one label (e.g. always missing hot_take but getting reaction right).
- **Confusion matrix** — shows exactly which labels are getting mixed up with each other. I expect the hardest boundary to be hot_take vs analysis, so I'll specifically look at how often those two get swapped.

Accuracy alone isn't enough here because a model that just predicts "reaction" for everything would get ~38% accuracy given my distribution — that's not a useful classifier. F1 per class tells me whether the model is actually learning the distinctions.

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
I used Claude to generate boundary posts between hot_take and analysis — the hardest pair in my taxonomy. Examples like "Iraq made so many fatal errors, 1 error led to goal 1, another led to goal 2" came from this process. Posts I couldn't classify cleanly forced me to tighten the decision rule: real analysis requires verifiable evidence, not just a breakdown of what happened.

**Annotation assistance:**
I used Claude to pre-label the full dataset of 261 examples using my label definitions and decision rules. I reviewed the distribution and spot-checked ~30 rows manually to verify the labels made sense. All pre-labeled examples are disclosed here — the full dataset was AI-assisted with human review.

**Failure analysis:**
After running the model, I'll feed the list of wrong predictions to Claude and ask it to identify patterns (e.g. "does it consistently confuse short hot_takes with reactions?" or "does it fail on quotes from experts?"). I'll verify any patterns it identifies by manually reviewing those examples myself before writing up the evaluation.
