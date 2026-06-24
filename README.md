# r/nba Discourse Classifier

A fine-tuned DistilBERT model that classifies r/nba Reddit posts into four discourse types: **Analytical Argument**, **Prediction**, **Hot Take**, and **Reaction/Joke/Meme**.

---

## Community and Task

**Community:** r/nba on Reddit — one of the largest sports communities online, with hundreds of thousands of active members who regularly debate trades, player rankings, team performance, and game outcomes.

**Why this community:** r/nba discourse spans a wide quality range within a single topic domain. The same event — say, a player's poor performance — generates careful statistical breakdowns, unsupported hot takes, season-end predictions, and pure emotional reactions, all in the same thread. That variation within a narrow topic makes it a well-suited classification task: the signal is in discourse structure and evidence use, not topic.

**Why these distinctions matter:** Members of r/nba actively police discourse quality — users frequently call out "hot takes" vs. "actual analysis." A classifier that distinguishes these categories could be useful for measuring discussion quality over time, surfacing well-reasoned posts, or flagging low-information content.

---

## Labels

| Label | Definition |
|---|---|
| **Analytical Argument** | A present-tense or historical claim supported by reasoning, statistics, evidence, or structured comparison. The reasoning would stand on its own without a forecast. |
| **Prediction** | A forward-looking claim about a future outcome — championship results, award winners, player development, season results. May or may not include reasoning; the label is determined by temporal orientation, not argument quality. |
| **Hot Take** | A strong present-tense or historical evaluative claim stated without supporting evidence. Bold assertion, no argument. |
| **Reaction/Joke/Meme** | A post that expresses a feeling — frustration, hype, sarcasm, humor — rather than making a claim. Reactive rather than argumentative. |

**Key decision boundary:** Prediction vs. Hot Take is decided by temporal orientation (future vs. present/historical). Hot Take vs. Analytical Argument is decided by whether supporting reasoning is present. Hot Take vs. Reaction is decided by whether the post makes a *claim* vs. expresses a *feeling*.

---

## Data

**Source:** 200 posts and comments collected from r/nba via the Arctic Shift Reddit archive API (public, unauthenticated). Drawn from game threads, post-game threads, season prediction threads, and player comparison discussions.

**Label distribution:**

| Label | Count | Share |
|---|---|---|
| Analytical Argument | 90 | 45.0% |
| Hot Take | 50 | 25.0% |
| Reaction/Joke/Meme | 45 | 22.5% |
| Prediction | 15 | 7.5% |

**Known imbalance:** Prediction is underrepresented at 7.5%. Many posts that initially appeared forward-looking were reclassified as Hot Takes after applying the temporal orientation rule strictly. The 15 remaining Prediction examples are high-confidence. This imbalance limits the model's ability to learn the Prediction class reliably and is reflected in the results.

**Split:** 70% train (139), 15% validation (31), 15% test (30). Splits use stratified sampling with random seed 42.

---

## Methodology

**Model:** `distilbert-base-uncased` fine-tuned for 4-class sequence classification.

**Hyperparameters:** 10 epochs, learning rate 5e-5, batch size 16, max sequence length 256.

**Deviation from notebook defaults (3 epochs, 2e-5 lr, no class weighting):** The default settings caused majority-class collapse — the model predicted "Analytical Argument" for all 30 test examples (accuracy 46.7%, macro F1 0.159). Two changes fixed this:
1. **Class-weighted loss:** Prediction received a weight of 3.16×, Reaction 1.12×, Hot Take 0.99×, Analytical Argument 0.56×. This penalizes the model more for missing minority-class examples.
2. **Longer training and higher LR:** 10 epochs at 5e-5 gave the model enough gradient updates to learn minority-class patterns without overfitting on the limited data.

---

## Evaluation Results

### Overall

| | Baseline (majority class) | Fine-tuned DistilBERT |
|---|---|---|
| **Accuracy** | 46.7% | 43.3% |
| **Macro F1** | 0.159 | **0.435** |

Accuracy is slightly lower for the fine-tuned model because it stopped predicting the majority class for everything — trading raw accuracy for balanced class learning. Macro F1 nearly tripled, which is the meaningful metric for this imbalanced task.

### Per-Class Metrics (Fine-tuned)

| Label | Precision | Recall | F1 | Test n |
|---|---|---|---|---|
| Analytical Argument | 0.500 | 0.429 | 0.462 | 14 |
| Hot Take | 0.400 | 0.571 | 0.471 | 7 |
| Prediction | 0.500 | 0.500 | 0.500 | 2 |
| Reaction/Joke/Meme | 0.333 | 0.286 | 0.308 | 7 |

Note: Prediction F1 (0.500) is based on 2 test examples and is not a reliable estimate.

### Confusion Matrix (Fine-tuned)

Rows = true label, columns = predicted label.

| | → Analytical Arg | → Hot Take | → Prediction | → Reaction/Joke/Meme |
|---|---|---|---|---|
| **Analytical Arg** (14) | **6** | 4 | 1 | 3 |
| **Hot Take** (7) | 2 | **4** | 0 | 1 |
| **Prediction** (2) | 1 | 0 | **1** | 0 |
| **Reaction/Joke/Meme** (7) | 3 | 2 | 0 | **2** |

---

## Failure Analysis

### AI-assisted pattern identification

Before writing this section, all 17 wrong predictions were reviewed for common themes. Three clear patterns emerged:

**Pattern 1 — Reaction/Joke/Meme is the hardest boundary (accounts for 9 of 17 errors):** The model struggles in both directions. It predicts Analytical Argument or Hot Take for Reaction posts (false negatives), and predicts Reaction for Analytical Argument posts (false positives). The underlying issue is that long, emotionally-charged posts that use analytical-sounding language look like arguments to the model, and brief analytical comparisons can look like reactions.

**Pattern 2 — Hot Take / Analytical Argument confusion (5 of 17 errors):** The model cannot reliably detect the presence or absence of supporting evidence. Posts that cite statistics can still be Hot Takes (if the stat is cherry-picked or decorative); posts that build a careful argument without hard numbers can be Analytical Arguments. The model has no reliable surface signal for "genuine reasoning."

**Pattern 3 — Data quality issues (at least 3 errors):** Several wrong predictions reflect mislabeled or off-topic examples in the pre-labeled dataset that were not corrected during review. For example, a post about Taco Bell was labeled Analytical Argument and a decontextualized sentence fragment was labeled Reaction/Joke/Meme. These errors directly caused wrong predictions and were visible in hindsight.

### 3 Analyzed Failures

**Failure 1 — Hot Take predicted as Analytical Argument**

> *"I know this is a hot, hot take, but this might have been the best NBA season of Victor's career. Unanimous DPOY, all-star, third in MVP voting, and made it to the Finals."*
> True: Hot Take | Predicted: Analytical Argument

The post explicitly frames itself as a hot take but then cites specific, verifiable evidence: DPOY award, All-Star selection, MVP voting finish, and Finals appearance. The model correctly identified the evidence and predicted Analytical Argument. This is arguably a **labeling error** — by the annotation rules, a post with supporting evidence should be Analytical Argument regardless of its framing. The inconsistency between the post's self-labeling and its content confused both the human annotator and reveals a gap in the decision rules: the rule "if evidence is present, use Analytical Argument" was not applied strictly enough when a post explicitly calls itself a hot take.

**Failure 2 — Analytical Argument predicted as Hot Take**

> *"Mavs are the most threatening in the paint, actually. Luka and Kyrie are great at beating any defender at the perimeter, which opens lanes and creates advantages for their bigs."*
> True: Analytical Argument | Predicted: Hot Take (conf: 0.577)

The post builds a specific causal argument: Luka and Kyrie's perimeter scoring creates lane-opening advantages for interior players. This is a structured basketball argument with no statistics, only positional reasoning. The model predicted Hot Take, likely because the evaluative framing ("most threatening," "great at") sounds like an assertion. **This is a boundary problem:** the Hot Take / Analytical Argument distinction depends on detecting *structural reasoning*, not just the presence of statistics. The model appears to have learned "statistics = Analytical Argument" and "evaluative adjectives without numbers = Hot Take," which is a shortcut that fails when the reasoning is qualitative rather than quantitative.

**Failure 3 — Analytical Argument predicted as Reaction/Joke/Meme**

> *"I think it goes without saying that he absolutely maximized the teams and talents he had and coached the team to 4 titles. Does he want to take on the next challenge of rebuilding back to greatness..."*
> True: Analytical Argument | Predicted: Reaction/Joke/Meme (conf: 0.643)

The phrase "I think it goes without saying" signals hedging or conversational tone — the kind of language common in reactions. The model appears to have picked up on this register and predicted Reaction despite the post making a substantive argument about a coach's track record. This is a **register problem**: the model has learned to associate casual, first-person language with Reaction, but casual language appears in Analytical Arguments too, especially in comment replies. The fix would require more training examples of Analytical Arguments written in a conversational register.

---

## Sample Classifications

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "The NBA Cup will catch on and become a big deal." | Prediction | **Prediction** ✓ | 0.652 |
| "The NBA Draft should be incredible TV. Instead, the league keeps botching it..." | Hot Take | **Hot Take** ✓ | 0.604 |
| "The timbre of this forum has shifted since the Knicks took it all and I'm all here for it..." | Reaction/Joke/Meme | **Reaction/Joke/Meme** ✓ | 0.596 |
| "I think it goes without saying that he absolutely maximized the teams and talents he had..." | Analytical Argument | Reaction/Joke/Meme ✗ | 0.643 |
| "Mavs are the most threatening in the paint, actually. Luka and Kyrie are great at beating..." | Analytical Argument | Hot Take ✗ | 0.577 |

**Why the Prediction example is reasonable:** "The NBA Cup will catch on and become a big deal" is a short, unambiguous future forecast with no present-tense claim. The model assigned it 65.2% confidence for Prediction and only 21.3% for Analytical Argument — correctly recognizing the temporal signal without any other surface features to work from.

---

## Reflection: What the Model Captured vs. What Was Intended

The label definitions were designed around *discourse structure*: the distinctions between claim-with-evidence, claim-without-evidence, future-oriented claim, and feeling-expression. What the model appears to have learned instead is a mix of surface proxies:

- **Statistics and numerical language** → Analytical Argument
- **Evaluative adjectives without numbers** → Hot Take
- **Future-tense verb phrases** → Prediction
- **Short posts or casual register** → Reaction/Joke/Meme

These proxies capture the easy cases but fail at the boundaries. A Hot Take that cites one cherry-picked stat gets classified as Analytical Argument. A structured qualitative argument gets classified as Hot Take. A conversational-register argument gets classified as Reaction.

The deeper issue is that the intended boundaries require *pragmatic understanding* — reasoning about why someone included a statistic, whether a claim is being used to build an argument or to assert a conclusion, whether emotional language is the whole point or just the framing around real basketball analysis. With 139 training examples and a model that has no prior basketball knowledge, this kind of reasoning is not learnable. The model learned shortcuts that work on the training distribution but break on the hard cases that matter most.

If the goal were deployment in a real community tool, the most important change would be more examples of the hard cases: Analytical Arguments written in emotional registers, Hot Takes that cite superficial statistics, and Reaction posts that embed claims. These are the examples that teach the model the *difference* rather than the average pattern of each class.

---

## Spec Reflection

**Where the spec helped:** The planning document's requirement to define a decision rule for every label pair before annotating forced a clear articulation of the Hot Take / Analytical Argument distinction (evidence present vs. absent) and the Prediction / Hot Take distinction (future vs. present orientation). Without those rules written down, annotation would have been far less consistent and the training signal would have been noisier.

**Where implementation diverged from the spec:** The spec planned for a balanced 50-per-label distribution. The actual distribution ended with Prediction at 7.5% because applying the temporal orientation rule strictly reclassified many apparent predictions as Hot Takes. Rather than collecting more examples to hit the target, the deviation was addressed with class-weighted loss — a modeling fix rather than a data fix. This was the right call given time constraints, but the result is that Prediction remains the least reliable class in the final model.

---

## AI Usage

**1. Data collection and pre-labeling (primary use):** Claude was used to fetch 308 r/nba posts and comments from the Arctic Shift Reddit archive API, apply pre-labels to all examples using the label definitions and annotation rules from planning.md, and flag low-confidence cases for closer human review. Each pre-label included a one-sentence reason. Overrides made during review included: relabeling several "Prediction" items that lacked future-tense signals to "Hot Take," removing off-topic items (geography, video game posts, ticket sales), and flagging decontextualized fragments for exclusion. The `ai_prelabel` column in `labeled_dataset.csv` records the original AI suggestion for every row.

**2. Failure pattern analysis:** After evaluating the fine-tuned model, all 17 wrong predictions were provided to Claude with the question: "What patterns do these share — similar post length, label pair confusion, sarcasm, or other features?" Claude identified the three patterns documented above (Reaction boundary failures, Hot Take / Analytical Argument confusion, data quality issues). Each pattern was then verified manually by re-reading the examples. One suggested pattern — "the model confuses sarcasm with Hot Takes" — was discarded after review: only one example had obvious sarcasm, and the other errors in the Reaction bucket were not sarcasm-driven.

**3. Label stress-testing (Milestone 2):** Before annotation, Claude generated 8 boundary posts for each label pair. Two posts that could not be cleanly assigned were used to tighten the decision rules in planning.md — specifically the rule that present-tense evaluative language plus one statistic does not automatically qualify as an Analytical Argument (it needs to be *reasoning*, not a single data point used decoratively).

---

## Demo

*[Demo video link — to be added after recording.]*

The demo shows 5 posts being classified with label and confidence score, narration of one correct and one incorrect prediction, and a walkthrough of this evaluation report.
