# TakeMeter — r/soccer Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in the r/soccer subreddit, distinguishing between analytical posts, hot takes, and reactions. Built for AI201 Project 3.

---

## Community Choice

I chose r/soccer because it's one of the most active sports communities on the internet with over 8 million members. Posts range from stat posts and historical records to heated opinions and fan reactions — exactly the kind of variation needed for a classification task. The three labels I defined (analysis, hot_take, reaction) map onto distinctions the community itself makes constantly: calling out bad takes and praising good analysis is part of how r/soccer operates.

---

## Label Taxonomy

**analysis** — The post supports a football claim using statistics, historical comparison, verifiable records, or tactical reasoning with real evidence.

- Example 1: "Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions (2006, 2010, 2014, 2018, 2022 & 2026)." → Specific, verifiable historical record.
- Example 2: "Curaçao's Eloy Room recorded 15 saves against Ecuador, the most on record (since 1966) by any goalkeeper in a FIFA World Cup match that did not feature extra-time." → Verifiable stat with historical context.

**hot_take** — A strong, confident football opinion stated with little or no supporting evidence — the post asserts rather than argues.

- Example 1: "Palmer is already world class." → Bold claim, no evidence or reasoning.
- Example 2: "France midfielders are painfully mediocre." → Confident opinion, no stats or argument behind it.

**reaction** — An emotional or observational response to a football event where the primary purpose is expressing feeling rather than making an argument.

- Example 1: "Marquinhos comforts his international teammate Gabriel after his penalty miss" → Describing an emotional moment, no argument.
- Example 2: "Cape Verde goalkeeper Vozinha after the draw vs Spain: 'I worked all my life for this. For this moment, for this dream.'" → Pure emotional response to a result.

---

## Data Collection

**Source:** r/soccer — post titles from the front page, comments from match threads (France vs Iraq, Norway vs Senegal), and the Daily Discussion thread.

**Labeling process:** Data was collected by saving r/soccer pages as text files and extracting post titles and comments. Claude was used to pre-label the full dataset using the label definitions from planning.md. I reviewed the distribution and spot-checked approximately 30 rows manually to verify labels were consistent with my definitions. All AI-assisted labeling is disclosed in the AI Usage section.

**Label distribution:**
| Label | Count |
|-------|-------|
| reaction | 98 |
| hot_take | 95 |
| analysis | 68 |
| **Total** | **261** |

**3 difficult-to-label examples:**

1. *"Thierry Henry in his post-match analysis about Ronaldo after Portugal's game vs DR Congo: 'The team needs to score, not you need to score.'"* — Could be analysis (expert making a tactical point) or hot_take (pundit opinion with no evidence). Labeled **hot_take** — if you remove Henry's credentials, there's no supporting argument left.

2. *"Iraq made so many fatal errors. 1 error led to goal 1, another error led to goal 2."* — Could be analysis (tactical breakdown) or hot_take (surface-level observation). Labeled **hot_take** — the reasoning is too vague to count as real analysis.

3. *"Anyway, his situation is morbidly funny. 3 UCLs between his 2 clubs in 3 years and him not winning any."* — Could be reaction (expressing amusement) or analysis (using a stat to make a point). Labeled **analysis** — the specific numbers are the substance of the point, not decoration for a feeling.

---

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:** Fine-tuned on Google Colab using a free T4 GPU. Training took approximately 22 seconds for 3 epochs on 182 training examples.

**Hyperparameters:**
- Epochs: 3
- Learning rate: 2e-5
- Batch size: 16

**Hyperparameter decision:** I kept the default 3 epochs rather than increasing them because validation loss was still decreasing at epoch 3 (0.975 vs 1.075 at epoch 2), suggesting the model hadn't overfit yet. With only 182 training examples, running more epochs risked overfitting.

**Train/validation/test split:** 70% / 15% / 15% (182 train, 39 validation, 40 test) — handled automatically by the notebook.

---

## Baseline Description

**Model:** Groq's `llama-3.3-70b-versatile` (zero-shot, no task-specific training)

**Prompt used:**
```
You are classifying posts from the r/soccer subreddit into exactly one of three categories:

- analysis: The post supports a football claim using statistics, historical comparison, verifiable records, or tactical reasoning with real evidence.
- hot_take: A strong, confident football opinion stated with little or no supporting evidence — the post asserts rather than argues.
- reaction: An emotional or observational response to a football event where the primary purpose is expressing feeling rather than making an argument.

Post: {text}

Respond with only one word — exactly one of: analysis, hot_take, reaction
```

All 40 test examples returned parseable responses (0 unparseable).

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 0.700 |
| Fine-tuned DistilBERT | 0.650 |

The baseline outperformed the fine-tuned model by 5 percentage points. This is discussed in the reflection section.

### Per-Class Metrics — Baseline (Groq)

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.88 | 0.70 | 0.78 | 10 |
| hot_take | 0.86 | 0.40 | 0.55 | 15 |
| reaction | 0.60 | 1.00 | 0.75 | 15 |
| **accuracy** | | | **0.70** | 40 |

### Per-Class Metrics — Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| analysis | 0.00 | 0.00 | 0.00 | 10 |
| hot_take | 0.92 | 0.73 | 0.81 | 15 |
| reaction | 0.54 | 1.00 | 0.70 | 15 |
| **accuracy** | | | **0.65** | 40 |

### Confusion Matrix — Fine-Tuned Model

| | Predicted: analysis | Predicted: hot_take | Predicted: reaction |
|---|---|---|---|
| **True: analysis** | 0 | 1 | 9 |
| **True: hot_take** | 0 | 11 | 4 |
| **True: reaction** | 0 | 0 | 15 |

The model never predicted analysis. Every analysis example was classified as either hot_take or reaction.

### 3 Wrong Predictions — Analysis

**Wrong prediction #1**
- Text: "Deschamps has led France to 16 wins in 21 World Cup matches, matching Helmut Schon's all time record."
- True label: analysis | Predicted: reaction (confidence: 0.40)
- Analysis: This is a clear stat post with a historical comparison — exactly what analysis is defined as. The model predicted reaction, likely because the sentence structure ("Deschamps has led France...") resembles the factual, news-style phrasing common in reaction posts like goal clip titles and transfer announcements. The model appears to have learned surface phrasing patterns rather than whether evidence is being used to support a claim.

**Wrong prediction #2**
- Text: "10 men and a statue: Portugal are sacrificing another World Cup for Cristiano Ronaldo's ego."
- True label: hot_take | Predicted: reaction (confidence: 0.45)
- Analysis: This is a strong opinion with a memorable framing ("10 men and a statue") but no supporting evidence — a classic hot_take. The model predicted reaction, probably because the post reads as an emotional outburst about a specific ongoing situation (Portugal's World Cup campaign). The boundary between an emotional hot_take and a reaction is genuinely hard: both are triggered by a specific event, and the model has apparently collapsed this distinction.

**Wrong prediction #3**
- Text: "Arsenal are the first ever team in Premier League history to go the whole season without receiving a red card or conceding a penalty"
- True label: analysis | Predicted: reaction (confidence: 0.46)
- Analysis: This is a verifiable historical record — a clear analysis label. The model predicted reaction with near-random confidence (0.46), suggesting it had no real signal to work with. The post has no emotional language or opinion framing at all, yet the model still missed it. This points to a fundamental failure to learn the analysis class: the model likely never developed a reliable representation of what makes a post analytical vs. reactive.

### Pattern Analysis

Of the 14 wrong predictions, 13 were misclassified as **reaction** — the model's dominant failure mode is predicting reaction when unsure. The analysis class was completely missed (F1 = 0.00): all 10 analysis examples in the test set were predicted as either reaction (9) or hot_take (1). This is a systematic collapse of one label, not random noise.

The likely causes are: (1) analysis is the smallest class (68 examples, 26% of training data) giving the model fewer examples to learn from; (2) analysis posts on r/soccer often resemble reaction posts in tone — they're still short, factual, and written in response to a specific event; (3) with only 182 training examples total, DistilBERT may not have had enough signal to learn the subtle distinction between a factual record post and an event-description post.

### Sample Classifications

| Text | True Label | Predicted | Confidence |
|------|-----------|-----------|------------|
| "France midfielders are painfully mediocre." | hot_take | hot_take | 0.91 |
| "Norway players and fans doing Viking Row" | reaction | reaction | 0.95 |
| "Mbappe is going to smash Messi's record" | hot_take | hot_take | 0.88 |
| "Deschamps has led France to 16 wins in 21 World Cup matches, matching Helmut Schon's all time record." | analysis | reaction | 0.40 |
| "Cape Verde goalkeeper Vozinha after the draw vs Spain: 'I worked all my life for this. For this moment, for this dream.'" | reaction | reaction | 0.93 |

**Correct prediction explained:** "France midfielders are painfully mediocre" → hot_take (confidence 0.91). This prediction is reasonable — the post uses strong evaluative language ("painfully mediocre") with no supporting evidence. The model has clearly learned to associate confident negative assessments with the hot_take label.

---

## Reflection: What the Model Learned vs. What I Intended

I intended the model to learn whether a post provides verifiable evidence for a claim. What it actually learned was a cruder distinction: **emotional/evaluative language (hot_take) vs. everything else (reaction)**. The analysis class — which requires detecting the presence of specific, verifiable evidence — was completely lost.

The model's decision boundary appears to be approximately: "does this post contain strong opinion language?" If yes → hot_take. If no → reaction. Analysis posts, which often lack opinion language entirely (they're just stating facts), fell into the reaction bucket because the model had no reliable signal for "this post contains verifiable evidence."

This is a labeling and data problem as much as a modeling problem. With only 68 analysis examples and many of them resembling reaction posts in surface form (short, factual, event-triggered), DistilBERT didn't have enough signal to learn the distinction. A fix would require more analysis examples that clearly differ from reaction posts in structure — longer posts, posts with explicit comparisons, posts that use numbers to argue a point rather than just report one.

---

## Spec Reflection

**One way the spec helped:** The spec's insistence on designing labels before collecting data was genuinely useful. Writing decision rules for edge cases before annotating 200 examples forced me to think through the analysis/hot_take boundary in concrete terms. When I encountered the Thierry Henry quote during annotation, I already had a rule to apply ("remove the speaker's credentials — does the argument survive?") rather than making an ad hoc call.

**One way implementation diverged:** The spec assumed manual annotation of 200 examples. I used Claude to pre-label the full dataset and reviewed the distribution rather than reading every example individually. This was faster but produced noisier labels — the analysis class in particular may have been inconsistently labeled because the pre-labeling relied on keyword patterns rather than genuine understanding of each post. A more careful manual annotation pass would likely have improved model performance on the analysis class.

---

## AI Usage

**Instance 1 — Dataset pre-labeling:** I provided Claude with my three label definitions and 261 unlabeled r/soccer posts and asked it to assign one label per post. Claude produced the initial labeled_data.csv. I reviewed the label distribution (reaction: 98, hot_take: 95, analysis: 68) and spot-checked approximately 30 rows manually. I overrode several labels — particularly on posts that Claude labeled as reaction when the post contained a verifiable stat (these should have been analysis). The pre-labeled dataset is the one used for training; all AI assistance is disclosed here.

**Instance 2 — Label stress-testing:** I asked Claude to generate 5–10 posts that sit at the boundary between hot_take and analysis to test whether my label definitions were precise enough. This produced examples like "Iraq made so many fatal errors — 1 error led to goal 1, another to goal 2." I couldn't classify this cleanly as written, which led me to tighten the decision rule: real analysis requires verifiable evidence, not just a description of what happened. I labeled this example hot_take and added the rule to planning.md before annotating.

**Instance 3 — Failure pattern analysis:** I pasted all 14 wrong predictions into Claude and asked it to identify common patterns. Claude identified that 13/14 wrong predictions were classified as reaction, and that the analysis class was completely missed. I verified this against the confusion matrix (0 correct analysis predictions) and used this pattern as the basis for my reflection section. I overrode Claude's suggestion that "short post length" was a factor — after re-reading the wrong predictions, I concluded the more accurate explanation was that analysis posts resemble reaction posts in surface form, not that they were short.
