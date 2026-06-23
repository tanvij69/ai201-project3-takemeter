# TakeMeter Project Planning

## Community

For this project, I will analyze discourse from the r/soccer subreddit, one of the largest football 
communities on the internet with over 8 million members. The community covers match discussions, 
transfer news, player performance debates, tactical analysis, and fan reactions from leagues and 
tournaments worldwide. Discourse ranges from detailed statistical and tactical reasoning to immediate 
emotional responses and unsupported bold opinions, making it a strong environment for discourse 
quality classification.

## Label Taxonomy

I am using 3 labels: **analysis**, **hot_take**, and **reaction**.

---

### Label 1: analysis

**Definition:**  
The post supports a football claim using tactical reasoning, statistics, historical comparison, or 
detailed football knowledge.

**Example 1 (from r/soccer):**  
"Cristiano Ronaldo becomes the first player to score a goal in six different FIFA World Cup editions 
(2006, 2010, 2014, 2018, 2022 & 2026)."  
→ Provides specific, verifiable historical data to support a claim about Ronaldo's career.

**Example 2 (from r/soccer):**  
"Curaçao's Eloy Room recorded 15 saves against Ecuador, the most on record (since 1966) by any 
goalkeeper in a FIFA World Cup match that did not feature extra-time."  
→ Uses a verifiable statistic with historical context to make a meaningful point.

**Uncertain Example:**  
"Spain's Head Coach Luis de la Fuente: 'Rodrigo is the best player in the world, and even at 50% 
he's much better than most midfielders in the world.'"  
→ Could be analysis (expert opinion from a coach) or hot_take (bold claim without supporting evidence).

**Decision Rule:**  
If the post provides specific, verifiable evidence or reasoning that would support the claim even 
without the opinion framing, label it analysis. If the claim is primarily bold and the evidence is 
thin or missing, label it hot_take.

---

### Label 2: hot_take

**Definition:**  
A strong, confident football opinion stated with little or no supporting evidence — the post asserts 
rather than argues.

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
If removing the opinion framing leaves behind a meaningful football argument with real evidence, 
label it analysis. If all that remains is the bold claim, label it hot_take.

---

### Label 3: reaction

**Definition:**  
An emotional response to a football event — a goal, a loss, a transfer, a referee decision — where 
the primary purpose is expressing feeling rather than making an argument.

**Example 1 (from r/soccer):**  
"WHAT A GOAL!!!"  
→ Pure emotional response to a goal clip.

**Example 2 (from r/soccer):**  
"Cape Verde goalkeeper Vozinha after the draw vs Spain: 'I worked all my life for this. For this 
moment, for this dream.'"  
→ Emotional response to a match result, no analytical content.

**Uncertain Example:**  
"I can't believe the manager subbed him off."  
→ Could be reaction (emotional frustration) or the start of a hot_take if the post goes on to make 
a claim.

**Decision Rule:**  
If the primary purpose is expressing a feeling in response to a specific event, label it reaction. 
If the post pivots into making a claim or argument, re-evaluate as hot_take or analysis.

---

## Hardest Edge Case

**Example:**  
"Thierry Henry in his post-match analysis about Ronaldo after Portugal's game vs DR Congo: 
'The team needs to score, not you need to score.'"

This could be:
- **analysis** — Henry is an expert making a tactical/strategic point about team vs. individual goals
- **hot_take** — it's a pundit quote with no data behind it, just a strong opinion

**Decision Rule:**  
Ask: "If I remove the speaker's name and credentials, is there still a meaningful argument here?"  
If yes → analysis. If it only works because of who said it → hot_take.  
The Henry quote → **hot_take** (the point is asserted, not argued with evidence).

---

## Why These Labels Matter

Members of r/soccer actively distinguish between posts that make real football arguments versus those 
that just state opinions confidently versus pure emotional reactions to events. These three categories 
reflect how the community itself evaluates discourse quality — calling out "this is just a hot take" 
or praising "actually good analysis" is common in comment sections.

---

## Data Collection Plan

- **Source:** r/soccer (post titles, flair text, and comment threads)
- **Target threads:** Match threads, Post-Match threads, Daily Discussion, transfer news posts
- **Goal:** 200+ labeled examples, approximately 67 per label
- **Split:** 70% train / 15% validation / 15% test
- **Format:** CSV with columns: `text`, `label`
