# Planning Document: r/nba Discussion Classification

## Community

### Chosen Community

I chose r/nba as the community for this classification project.

### Why This Community?

r/nba contains a wide range of basketball-related discussion styles. Users frequently post detailed analysis, predictions, controversial opinions, emotional reactions, jokes, and memes. Because discussion quality varies significantly, it is a good environment for a classification task that attempts to distinguish different types of discourse.

### Why This Is Interesting

Members of r/nba regularly debate whether a comment is thoughtful analysis or simply a hot take. These distinctions matter because users often value evidence-based discussion and criticize unsupported opinions. A classifier that recognizes these categories could be useful for moderation, discourse analysis, or content filtering.

---

# Labels

## Label 1: Analytical Argument

### Definition

A post that makes a basketball-related claim and supports it with reasoning, evidence, comparisons, statistics, or detailed explanation.

### Examples

1. "The Celtics' defense works because they can switch every position, which prevents opponents from creating mismatches."

2. "Luka's offensive production is elite, but his high usage rate makes roster construction more difficult because teammates have fewer opportunities to create."

---

## Label 2: Prediction / Forecast

### Definition

A post whose primary purpose is to forecast a specific future outcome — championship results, award winners, player development, career trajectories, or season finish. The distinguishing feature is temporal orientation: the post is primarily about what *will* happen, not what *is* or *has* happened. A prediction may or may not include supporting reasoning; the label is determined by whether a future outcome is being forecast, not by the quality of the argument.

### Examples

1. "Victor Wembanyama will win multiple MVP awards before he turns 30." *(unsupported forecast — still Prediction, not Hot Take, because it targets a future outcome)*

2. "Luka will never win a championship playing this way because his style limits roster flexibility and forces teams into bad contracts." *(supported forecast — still Prediction, not Analytical Argument, because the claim centers on a future outcome)*

---

## Label 3: Hot Take / Unsupported Opinion

### Definition

A strong evaluative claim that provides little or no supporting evidence or reasoning.

### Examples

1. "Kawhi Leonard is better than Kevin Durant."

2. "Steve Kerr is the most overrated coach in NBA history."

---

## Label 4: Reaction / Joke / Meme

### Definition

A post intended primarily as humor, emotional reaction, sarcasm, celebration, frustration, or social engagement rather than basketball analysis.

### Examples

1. "The Lakers are so cooked."

2. "Basketball is easy when the ball goes in the hoop."

---

# Hard Edge Cases

Some posts naturally sit near the boundary between two labels.

## Edge Case 1: Prediction vs Analytical Argument

### Example

"Luka will never win a championship playing this way because his style limits roster flexibility."

### Possible Labels

* Prediction
* Analytical Argument

### Decision

Prediction

### Reason

The claim centers on a future outcome (winning a championship). Even though reasoning is present, the post is forecasting what will happen — not explaining current dynamics for their own sake. Decision rule: if removing the future-tense claim would make the post pointless, it's a Prediction. An Analytical Argument stands on its own as an explanation of present or historical basketball; a Prediction uses reasoning in service of a forecast.

### Contrast Case (Analytical Argument)

"Luka's high usage rate makes roster construction more difficult because teammates have fewer opportunities to create." — This explains a current dynamic without centering a forecast. → Analytical Argument.

---

## Edge Case 2: Hot Take vs Analytical Argument

### Example

"Kyrie isn't a top-five point guard of the last twenty years."

### Possible Labels

* Hot Take
* Analytical Argument

### Decision

Hot Take

### Reason

No supporting evidence or explanation is provided.

---

## Edge Case 3: Reaction vs Hot Take

### Example

"This team is trash."

### Possible Labels

* Reaction / Joke / Meme
* Hot Take

### Decision

Reaction / Joke / Meme

### Reason

The statement is primarily emotional and does not attempt to justify its evaluation.

---

## Annotation Rules for Ambiguous Cases

Apply these rules in order — the first rule that matches wins:

1. If the post is primarily emotional, humorous, sarcastic, or reactive with no basketball argument, use **Reaction / Joke / Meme**.
2. If the post's primary claim is about a *future* event or outcome (championship, award, development, season result), use **Prediction** — regardless of whether reasoning is present.
3. If the post makes a present-tense or historical evaluative claim *with* supporting reasoning, evidence, or structured argument, use **Analytical Argument**.
4. If the post makes a present-tense or historical evaluative claim *without* supporting reasoning, use **Hot Take**.

Key distinctions:
- **Prediction vs. Hot Take**: temporal. Future-oriented → Prediction. Present/historical → Hot Take.
- **Prediction vs. Analytical Argument**: temporal. Future outcome as the main claim → Prediction. Current explanation as the main claim → Analytical Argument.
- **Hot Take vs. Reaction**: intent. A Hot Take makes a *claim* (asserting something is true). A Reaction expresses a *feeling* (frustration, joy, disbelief).

---

# Data Collection Plan

## Data Source

Examples will be collected from public r/nba posts and comments via manual browsing on Reddit. I will navigate directly to relevant threads, copy qualifying posts and top-level comments, and paste them into the dataset CSV by hand. No API or scraping tool will be used at this stage.

Thread types to draw from:

* Game threads
* Post-game threads
* Season prediction threads
* Player comparison discussions
* Team discussion threads
* General NBA discussion posts

Drawing from multiple thread types is intentional: game threads over-represent Reaction posts, prediction threads over-represent Forecasts, and comparison discussions over-represent Hot Takes and Analytical Arguments. Mixing thread types is necessary to hit the balanced 50-per-label target without artificially skewing the distribution.

---

## Dataset Size

Target dataset size: 200 labeled examples.

### Target Distribution

| Label                  | Target Count |
| ---------------------- | ------------ |
| Analytical Argument    | 50           |
| Prediction             | 50           |
| Hot Take               | 50           |
| Reaction / Joke / Meme | 50           |

Total: 200 examples

A balanced dataset should improve model performance and simplify evaluation.

---

## Handling Imbalance

If one label is underrepresented after collecting 200 examples:

1. Search specifically for thread types where that label appears more frequently.
2. Continue collecting until every class has at least 40 examples.
3. Document any remaining imbalance in the final report.

No label should exceed 70% of the dataset.

---

# Evaluation Metrics

## Accuracy

Accuracy provides an overall measure of classifier performance.

However, accuracy alone is not sufficient because some classes may be easier to predict than others.

---

## Precision

Precision measures how often predictions for a class are correct.

This is important because incorrectly labeling a Hot Take as Analytical Argument would reduce the usefulness of the classifier.

---

## Recall

Recall measures how many examples of a class are successfully identified.

This is important because missing Analytical Arguments would reduce the classifier's ability to identify substantive discussion.

---

## Macro F1 Score

Macro F1 treats all classes equally and balances precision and recall across all four labels.

This is the primary metric for two reasons. First, the target distribution is balanced (50 examples per class), so equal weighting is appropriate — no class dominates the dataset in a way that would make per-class weighting more informative. Second, all four discourse types are equally valuable to identify: a classifier that is excellent at finding Reactions but poor at finding Analytical Arguments is not useful for discourse quality analysis, even if its overall accuracy looks acceptable.

---

## Confusion Matrix

A confusion matrix will identify which labels are frequently confused.

Expected confusion areas include:

* Prediction vs Analytical Argument
* Hot Take vs Reaction / Joke / Meme

---

# Definition of Success

The classifier will be considered useful if it achieves:

* Accuracy ≥ 75%
* Macro F1 ≥ 0.70
* No individual class F1 below 0.60

The classifier will be considered strong enough for deployment in a community-analysis tool if it achieves:

* Accuracy ≥ 85%
* Macro F1 ≥ 0.80
* No individual class F1 below 0.70

These thresholds provide objective criteria for success.

---

# AI Tool Plan

## 1. Label Stress-Testing

Before annotation, I will provide my label definitions and edge cases to an AI tool and ask it to generate 5–10 examples that sit between two labels.

Examples include:

* Prediction vs Analysis
* Analysis vs Hot Take
* Hot Take vs Reaction

If generated examples cannot be classified consistently, I will revise my definitions before collecting data.

---

## 2. Annotation Assistance

I will use Claude to generate preliminary labels for each batch of collected examples before reviewing them myself. Using AI pre-labels speeds up annotation and surfaces cases where the model's reading differs from mine — those disagreements are often the most informative edge cases.

Workflow:

1. Collect a batch of 20–30 examples.
2. Submit each example to Claude with the full label definitions and annotation rules, asking for a label and a one-sentence reason.
3. Review every label manually, checking the stated reason against my own reading.
4. Override any label where I disagree. Record the final label in the `label` column.
5. Record the AI's suggestion in a separate `ai_prelabel` column (values: Analytical Argument, Prediction, Hot Take, Reaction/Joke/Meme, or blank if no pre-label was generated).

Final labels will always be determined through human review. The `ai_prelabel` column will be used in post-hoc analysis to measure how often the AI and human labels agreed, and whether disagreements cluster around specific label pairs.

---

## 3. Failure Analysis

After evaluation, I will collect all misclassified examples.

I will ask an AI tool to identify possible patterns such as:

* Ambiguous wording
* Overlap between labels
* Lack of context
* Consistent model weaknesses

I will manually verify every suggested pattern before including conclusions in the final report.

The AI tool will be used for hypothesis generation only.

---

# Planned Updates

This planning document may be updated if:

* Label definitions change after stress-testing.
* Additional edge cases are discovered.
* Data collection procedures change.
* Stretch features are added later.

All updates will be documented before implementation.
