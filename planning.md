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

A post whose primary purpose is predicting a future outcome rather than explaining a current one.

### Examples

1. "Victor Wembanyama will win multiple MVP awards before he turns 30."

2. "The Thunder will reach the NBA Finals within the next two seasons."

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

Analytical Argument

### Reason

The primary purpose is explaining why an outcome may occur rather than simply forecasting the future.

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

1. If substantial reasoning is present, use Analytical Argument.
2. If the focus is on a future outcome, use Prediction.
3. If a strong opinion is stated without support, use Hot Take.
4. If the comment is mainly emotional, humorous, sarcastic, or reactive, use Reaction / Joke / Meme.

---

# Data Collection Plan

## Data Source

Examples will be collected from public r/nba posts and comments.

Sources include:

* Game threads
* Post-game threads
* Season prediction threads
* Player comparison discussions
* Team discussion threads
* General NBA discussion posts

Using multiple thread types should improve dataset diversity.

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

Macro F1 treats all classes equally and balances precision and recall.

This is the primary metric because all four labels are important.

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

I may use an LLM to generate preliminary labels for a batch of examples.

Workflow:

1. Collect examples.
2. Generate AI label suggestions.
3. Manually review every label.
4. Correct mistakes before adding examples to the dataset.

A separate column called "AI_Prelabel" will indicate whether a label was initially suggested by AI.

Final labels will always be determined through human review.

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
