# TakeMeter — r/soccer Discourse Classifier

A fine-tuned text classifier that evaluates discourse quality in the r/soccer subreddit, distinguishing between analytical posts, hot takes, and reactions. Built for AI201 Project 3.

---

## Community Choice

There are many people participating in r/soccer over 8 million people. Because of this large size, there are many different kinds of posts, for example, stats and historical data, as well as very heated and opinionated posts made by fans. All these types of posts are good examples of the kinds of post variations I need for my classification task. In terms of posting types, I developed 3 categories (analysis, hot_take, reaction) based on how this subreddit works. Users regularly identify bad takes from others and provide positive feedback for good analysis.

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

**Hyperparameter decision:** I kept the default 3 epochs rather than increasing them because the validation loss was still decreasing at epoch 3 (0.975 vs. 1.075 at epoch 2), suggesting the model hadn’t overfit yet. With only 182 training examples, running more epochs would have increased the risk of overfitting.

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
- Analysis: This is a clear example of a stats post and a historical comparison and therefore is certainly what analysis is. The prediction of the reaction was likely due to the fact that the sentence structure used here may be similar to what you would typically see in reaction posts and goal clip titles and even transfer announcements (i.e., the typical news type sentence structure of past, ("Deschamps has led France...")). Therefore, it appears that the model/rule is based on surface type language and not whether there has actually been any evidence provided to support the claim being made.

**Wrong prediction #2**
- Text: "10 men and a statue: Portugal are sacrificing another World Cup for Cristiano Ronaldo's ego."
- True label: hot_take | Predicted: reaction (confidence: 0.45)
- Analysis: While this is a strong opinion with a memorable framing ("10 men and a statue"), it lacks supporting evidence and is therefore a classic hot_take. Based on how it appears to be an emotional outburst related to a current event (Portugal's World Cup campaign), the model predicted that it would generate responses. There does appear to be some difficulty distinguishing between an emotional hot_take and a reaction, both seem to get triggered by an event in common, but the model has apparently failed to differentiate between the two.

**Wrong prediction #3**
- Text: "Arsenal are the first ever team in Premier League history to go the whole season without receiving a red card or conceding a penalty"
- True label: analysis | Predicted: reaction (confidence: 0.46)
- Analysis: This is a factual historical record with a clear analysis label. The model predicted reaction nearly random amount of confidence (0.46), indicating that it had no signals to be able to predict the post being an analytical post. The post itself contains neither emotion nor opinion in how it is framed, yet still failed to predict the post as being analytical. This indicates that there is a problem with the model learning the distinction of an analytical post vs. a reactive post in general, or that the model hasn't been able to create a consistent representation of an analytical post vs. a reactive post.

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

**Instance 1 — Dataset pre-labeling:** I gave Claude a set of three label definitions along with 261 unlabeled r/soccer posts in order to assign each of the posts a single label. The result was the initial labeled_data.csv. After reviewing the initial distribution of labeled data (reaction: 98, hot_take: 95, analysis: 68) and sampling approximately thirty rows from the original data, I manually overrode several labels particularly on posts that Claude labeled as reaction when the post contained a verifiable stat. The pre-labeled dataset is the training set; there is a complete disclosure of the use of AI in this process.

**Instance 2 — Label stress-testing:** I asked Claude to create me between five and ten posts that would represent metaphorically "the hot take and an analysis" to use to evaluate how clear my label definitions were. One of the posts produced was Iraq made so many fatal errors — 1 error led to goal 1, another to goal 2." I could not classify as written so I revised my decision rules for real analysis as in order to be an analysis there must be verifiable proof and not only a description of events. I labeled this hot_take and added the rule into planning.md prior to proceeding with annotating.

**Instance 3 — Failure pattern analysis:** I pasted all 14 wrong predictions into Claude and asked it to identify common patterns. Claude identified that 13/14 wrong predictions were classified as reaction, and that the analysis class was completely missed. I verified this against the confusion matrix (0 correct analysis predictions) and used this pattern as the basis for my reflection section. I overrode Claude's suggestion that "short post length" was a factor — after re-reading the wrong predictions, I concluded the more accurate explanation was that analysis posts resemble reaction posts in surface form, not that they were short.
