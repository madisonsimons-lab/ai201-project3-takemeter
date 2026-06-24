# Planning Document: r/nba Discussion Classification

## Community Selection

### Chosen Community

I chose the Reddit community r/nba.

### Why This Community?

r/nba is one of the largest sports discussion communities on Reddit and contains a wide range of discussion styles. Users post detailed basketball analysis, predictions about future outcomes, controversial opinions, reactions to games, memes, and short emotional responses. The variety of discourse makes it a strong candidate for a classification task because posts differ not only in topic but also in purpose and quality.

### Why This Is Interesting

Members of the community regularly distinguish between thoughtful analysis and low-effort content. Discussions often include disagreements about whether a claim is supported by evidence or is simply a "hot take." A classifier that can identify these different discussion styles could be useful for content filtering, moderation assistance, or discourse-quality analysis.

---

# Label Definitions

## Label 1: Analytical Argument

### Definition

A post that makes a basketball-related claim and supports it with reasoning, evidence, comparisons, statistics, or detailed explanation.

### Examples

1. "The Celtics' defense works because they can switch almost every position, which prevents teams from exploiting mismatches in playoff series."
2. "Luka's usage rate creates elite offensive production, but it can also make roster construction more difficult because teammates have fewer opportunities to create."

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

## Prediction vs. Analytical Argument

Example:
"Luka will never win a championship playing this way because his style limits roster flexibility."

This post contains both prediction and analysis.

### Annotation Rule

Classify based on the primary purpose of the post.

* If the post focuses mainly on explaining why something happens, label it Analytical Argument.
* If the post focuses mainly on forecasting a future outcome, label it Prediction.

---

## Hot Take vs. Analytical Argument

Example:
"Kyrie is not a top-five point guard of the last twenty years."

### Annotation Rule

* If reasoning or evidence is provided, label as Analytical Argument.
* If the statement is presented without support, label as Hot Take.

---

## Reaction vs. Hot Take

Example:
"This team is trash."

### Annotation Rule

* If the statement is primarily emotional or reactive, label as Reaction.
* If it is presented as a serious evaluation or ranking claim, label as Hot Take.

---

# Data Collection Plan

## Data Source

Data will be collected from public r/nba posts and comments.

I will collect examples from:

* Game threads
* Post-game threads
* Discussion threads
* Opinion threads
* Player comparison discussions
* Season prediction discussions

Collecting from multiple thread types should improve diversity and reduce sampling bias.

## Dataset Size

Target dataset size: 200 labeled examples.

Target distribution:

| Label               | Target Count |
| ------------------- | ------------ |
| Analytical Argument | 50           |
| Prediction          | 50           |
| Hot Take            | 50           |
| Reaction/Joke/Meme  | 50           |

Balanced classes will simplify training and evaluation.

## If Labels Are Underrepresented

Prediction and Analytical Argument posts may be less common than reactions.

If a label is underrepresented after collecting 200 examples:

1. Search specifically for threads where that label is common.
2. Continue collecting until at least 40 examples exist for every class.
3. Document any remaining class imbalance in the final report.

---

# Evaluation Metrics

## Accuracy

Accuracy measures overall classification performance and provides an easy-to-understand baseline.

However, accuracy alone is insufficient because some classes may be easier to predict than others.

## Precision

Precision measures how often a predicted label is correct.

This is important because incorrectly labeling a Hot Take as Analytical Argument would reduce the usefulness of the classifier.

## Recall

Recall measures how many examples of a class are successfully identified.

This matters because missing Analytical Arguments would reduce the classifier's ability to identify substantive discussion.

## F1 Score

F1 combines precision and recall into a single metric.

Macro-averaged F1 will be used because all classes are important and should contribute equally to evaluation.

## Confusion Matrix

A confusion matrix will help identify which labels are frequently mistaken for one another.

This is particularly important because:

* Prediction and Analysis may overlap.
* Hot Takes and Reactions may overlap.

---

# Definition of Success

The classifier will be considered useful if it achieves:

* Accuracy ≥ 75%
* Macro F1 ≥ 0.70
* No individual class F1 below 0.60

The classifier will be considered deployment-ready for a community analysis tool if it achieves:

* Accuracy ≥ 85%
* Macro F1 ≥ 0.80
* No individual class F1 below 0.70

These thresholds are specific enough to determine objectively whether the project succeeds.

---

# AI Tool Plan

## 1. Label Stress-Testing

Before annotation begins, I will provide the label definitions and edge cases to an AI tool and ask it to generate 5–10 examples that sit between two labels.

Examples:

* Prediction vs Analysis
* Analysis vs Hot Take
* Hot Take vs Reaction

If generated examples cannot be classified consistently, I will revise the definitions before collecting data.

Goal:
Reduce ambiguity before annotating 200 examples.

---

## 2. Annotation Assistance

I will use an LLM to suggest preliminary labels for some examples.

Workflow:

1. Collect raw posts/comments.
2. Generate AI label suggestions.
3. Review every label manually.
4. Correct mistakes before adding examples to the dataset.

Tracking:
A separate spreadsheet column called "AI_Prelabel" will indicate whether a label was initially suggested by AI.

The final label will always be determined by human review.

---

## 3. Failure Analysis

After evaluating the classifier, I will collect all misclassified examples.

I will ask an AI tool to identify possible patterns such as:

* Overlap between Prediction and Analysis
* Overlap between Reaction and Hot Take
* Ambiguous wording
* Insufficient context

I will then manually verify every suggested pattern using the actual misclassified examples before including conclusions in the final report.

The AI tool will be used to generate hypotheses, not final conclusions.

---

# Planned Updates

This planning document will be updated if:

* Label definitions change after stress-testing.
* Additional classes are introduced.
* Data collection procedures change.
* Stretch features are added later in the project.

Any changes will be documented before implementation or additional annotation begins.

