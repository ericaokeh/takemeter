# takemeter

# TakeMeter

## Project Overview

TakeMeter is a text-classification project built around comments from the Reddit community **r/AmIOverreacting**.

The goal is to classify whether a commenter supports the original poster's reaction or critiques how the original poster reacted.

Rather than classifying the topic or general sentiment of a comment, the model focuses on a narrower question:

**Does the commenter believe the original poster's reaction was reasonable, or do they believe the reaction was excessive, disproportionate, or handled poorly?**

This project compares a zero-shot baseline using **Groq with `openai/gpt-oss-120b`** against a fine-tuned **DistilBERT (`distilbert-base-uncased`)** classifier.

---

## Community and Dataset

The dataset was collected from **r/AmIOverreacting**, a subreddit where users describe interpersonal situations and ask whether their reaction was reasonable.

The unit of analysis is the **comment**, not the original Reddit post.

The final labeled dataset contains:

* **200 comments**
* **120 `SUPPORTS_REACTION` examples**
* **80 `CRITIQUES_REACTION` examples**
* A **60/40 class distribution**
* Comments collected across multiple Reddit threads
* Deleted, removed, duplicate, and OP-authored comments excluded
* Obvious subreddit verdict tokens such as `NOR` and `YOR` removed from model-training text to reduce lexical leakage

The dataset was split using a stratified 70/15/15 split:

* **Training:** 140 examples
* **Validation:** 30 examples
* **Test:** 30 examples

Stratification preserved the overall label balance across each split.

---

## Label Taxonomy

### `SUPPORTS_REACTION`

The commenter believes the original poster's reaction, feelings, or response is reasonable or justified.

Example:

> "You already told him that bothered you. You have every right to be upset."

### `CRITIQUES_REACTION`

The commenter believes the original poster is overreacting, responding too strongly, misinterpreting the situation, or handling it poorly.

Example:

> "Being upset makes sense, but ending the friendship over this is too much."

### Edge-Case Rule

The most important boundary in the taxonomy is the difference between **validating someone's feelings** and **supporting what they actually did**.

For example:

> "You have every right to be angry, but breaking up over this is too much."

This is labeled `CRITIQUES_REACTION` because the commenter ultimately criticizes the action the original poster took.

Comments that were only jokes, questions, unrelated discussion, or advice without an interpretable judgment of the original poster's reaction were excluded.

---

## Data Collection and Annotation

Comments were collected from public Reddit threads in **r/AmIOverreacting**.

During annotation, each comment was reviewed according to the two-label taxonomy. The primary annotation question was:

**What is the commenter's overall judgment of the original poster's actual reaction?**

Comments containing mixed language required special attention. When a commenter validated the original poster's emotions but criticized the action taken, the final judgment of the action determined the label.

A small number of difficult examples were flagged during annotation because they contained mixed judgments or indirect language.

To reduce the chance that the classifier would simply memorize subreddit shorthand, explicit verdict tokens such as `NOR` and `YOR` were removed from the model-training text while preserving the rest of the comment.

---

## Modeling

### Fine-Tuned Model

The supervised classifier uses:

**`distilbert-base-uncased`**

DistilBERT was fine-tuned on the labeled Reddit comments using the training split and evaluated on the held-out test split.

The classification mapping was:

| Label                | ID |
| -------------------- | -: |
| `SUPPORTS_REACTION`  |  0 |
| `CRITIQUES_REACTION` |  1 |

### Zero-Shot Baseline

The zero-shot comparison uses:

**Groq — `openai/gpt-oss-120b`**

The model receives the same label definitions used during annotation and is instructed to return exactly one of the two valid label strings.

The baseline prompt explains:

* The Reddit community and classification task
* Both label definitions
* One example per label
* The mixed-judgment edge-case rule
* The requirement to output only the final label

---

# Evaluation

## Overall Accuracy

The held-out test set contains **30 examples**.

| Model                           |                       Accuracy |
| ------------------------------- | -----------------------------: |
| Zero-shot `openai/gpt-oss-120b` | **[INSERT BASELINE ACCURACY]** |
| Fine-tuned DistilBERT           |                       **0.60** |

The fine-tuned model correctly classified **18 of 30 test examples**.

The earlier baseline result returned `NaN` because the zero-shot predictions were not successfully parsed. That is not a valid accuracy result, so the final baseline accuracy should be inserted after the Groq classifier produces usable predictions.

---

## Per-Class Metrics

### Zero-Shot Baseline

| Label                | Precision |   Recall |       F1 |
| -------------------- | --------: | -------: | -------: |
| `SUPPORTS_REACTION`  |  [INSERT] | [INSERT] | [INSERT] |
| `CRITIQUES_REACTION` |  [INSERT] | [INSERT] | [INSERT] |

### Fine-Tuned DistilBERT

| Label                | Precision |   Recall |       F1 |
| -------------------- | --------: | -------: | -------: |
| `SUPPORTS_REACTION`  |  [INSERT] | [INSERT] | [INSERT] |
| `CRITIQUES_REACTION` |  [INSERT] | [INSERT] | [INSERT] |

Per-class results are important because overall accuracy alone cannot show whether the model disproportionately favors the larger `SUPPORTS_REACTION` class.

---

## Confusion Matrix

Insert the values produced by the notebook's confusion matrix.

| Actual ↓ / Predicted → | `SUPPORTS_REACTION` | `CRITIQUES_REACTION` |
| ---------------------- | ------------------: | -------------------: |
| `SUPPORTS_REACTION`    |            [INSERT] |             [INSERT] |
| `CRITIQUES_REACTION`   |            [INSERT] |             [INSERT] |

The confusion matrix helps reveal whether the errors are directional. For example, if many `CRITIQUES_REACTION` comments are incorrectly predicted as `SUPPORTS_REACTION`, that would suggest the model has learned supportive language more strongly than the boundary between emotional validation and criticism of an action.

---

## Error Analysis

The fine-tuned model achieved **60% accuracy**, meaning it misclassified **12 of the 30 test examples**.

Before writing the final error analysis, I used an AI tool to help surface recurring patterns across the misclassified examples. I asked it to look for patterns involving:

* Mixed or qualified judgments
* Indirect language
* Sarcasm
* Short or low-information comments
* Differences in comment length
* Validation of feelings followed by criticism of behavior
* Possible class imbalance
* Repeated confusion between the same labels

I then manually reviewed the misclassified comments to determine which suggested patterns were actually supported by the examples.

### Error Example 1

**Comment:**
[PASTE MISCLASSIFIED COMMENT]

**True label:** `[INSERT]`
**Predicted label:** `[INSERT]`

**Analysis:**
[Explain what language likely confused the model. Did the comment validate OP before criticizing the action? Was the judgment indirect? Was the comment too short to provide much evidence?]

**What could improve this:**
[For example: add more mixed-judgment examples to the training data or tighten the annotation rule.]

### Error Example 2

**Comment:**
[PASTE MISCLASSIFIED COMMENT]

**True label:** `[INSERT]`
**Predicted label:** `[INSERT]`

**Analysis:**
[INSERT YOUR ANALYSIS]

**What could improve this:**
[INSERT]

### Error Example 3

**Comment:**
[PASTE MISCLASSIFIED COMMENT]

**True label:** `[INSERT]`
**Predicted label:** `[INSERT]`

**Analysis:**
[INSERT YOUR ANALYSIS]

**What could improve this:**
[INSERT]

---

## AI-Assisted Error Pattern Analysis

The AI-assisted review was used as a pattern-finding tool rather than as the final evaluator.

One pattern I specifically looked for was whether the classifier struggled with comments that contain both supportive and critical language. These examples are difficult because a comment may contain words that sound supportive while its final judgment still criticizes the original poster's behavior.

For example:

> "You have a right to be upset, but ending the relationship over it is extreme."

This contains supportive language at the beginning, even though the final classification should be `CRITIQUES_REACTION`.

This distinction matters because the intended task is not ordinary positive-versus-negative sentiment classification. The model must identify the commenter's **judgment of the reaction itself**.

Any additional pattern suggested by the AI was manually checked against the actual errors before being included in the final analysis. Patterns that did not appear consistently across the misclassified examples were discarded rather than treated as meaningful findings.

---

## Sample Classifications

The following section should contain 3–5 examples passed through the fine-tuned model with the model's confidence score.

| Comment            | Predicted Label  | Confidence |
| ------------------ | ---------------- | ---------: |
| [INSERT COMMENT]   | `[INSERT LABEL]` |   [INSERT] |
| [INSERT COMMENT]   | `[INSERT LABEL]` |   [INSERT] |
| [INSERT COMMENT]   | `[INSERT LABEL]` |   [INSERT] |
| [OPTIONAL COMMENT] | `[INSERT LABEL]` |   [INSERT] |
| [OPTIONAL COMMENT] | `[INSERT LABEL]` |   [INSERT] |

For at least one correctly classified example, explain why the result is reasonable.

**Example explanation:**
A prediction of `SUPPORTS_REACTION` is reasonable when the commenter explicitly says that the original poster's response was justified and does not criticize what the original poster did.

---

# Reflection: What the Model Captured vs. What I Intended

The intended taxonomy requires the model to learn more than whether a Reddit comment sounds supportive or negative. It must identify the commenter's judgment of the **original poster's reaction**.

That distinction creates an important gap between the conceptual task and the patterns a text classifier can learn from a relatively small dataset.

The model may learn lexical shortcuts associated with agreement, disagreement, emotional validation, or criticism. However, those signals do not always map directly onto the intended labels.

For example:

> "I understand why you're angry, but what you did afterward was too much."

This contains emotional validation and criticism at the same time. The correct classification depends on recognizing that the commenter ultimately critiques the behavior. A model relying heavily on supportive phrases such as "I understand" or "you're right to be angry" may incorrectly predict `SUPPORTS_REACTION`.

This means the model's decision boundary may partially reflect **surface-level agreement language** rather than the deeper distinction I intended: whether the commenter believes the original poster's actual response was proportionate.

The relatively small training set also limits the number of examples available for subtle cases such as mixed judgments, sarcasm, indirect criticism, and qualified agreement.

A stronger version of the model would likely require:

* More examples of mixed supportive/critical comments
* More examples of indirect criticism
* Additional examples from the smaller `CRITIQUES_REACTION` class
* More explicit representation of difficult edge cases during training
* Continued review of annotation consistency

---

# Spec Reflection

## How the Spec Helped

The project specification helped guide the implementation by requiring a small, mutually exclusive label taxonomy before model training.

This forced me to define the distinction between `SUPPORTS_REACTION` and `CRITIQUES_REACTION` explicitly rather than labeling examples based on intuition alone. The requirement to document edge cases was especially useful because mixed comments quickly became one of the most difficult parts of the dataset.

The evaluation requirements also pushed the project beyond reporting a single accuracy score. Per-class metrics, a confusion matrix, and individual error analysis provide more information about what the classifier actually learned.

## How the Implementation Diverged

One implementation decision that went beyond the simplest version of the specification was removing explicit subreddit verdict tokens such as `NOR` and `YOR` from the training text.

Those tokens can directly reveal the intended judgment. Leaving them in the dataset could allow the model to learn a shortcut rather than learning the language surrounding whether a reaction is reasonable.

I therefore removed the obvious verdict tokens while preserving the rest of each comment. This made the classification task harder, but it better matched the behavior I wanted the model to learn.

---

# AI Usage

AI tools were used as assistance during the project, but final design and evaluation decisions were manually reviewed.

## 1. Taxonomy and Annotation Support

I used an AI assistant to stress-test the label definitions and identify difficult annotation cases.

I directed the tool to compare comments against the definitions of `SUPPORTS_REACTION` and `CRITIQUES_REACTION`, especially for comments that both validated the original poster's feelings and criticized the action they took.

The AI surfaced possible classifications and edge cases. I manually reviewed the examples and applied the final labels myself. I also overrode suggestions when they did not follow the project's final mixed-judgment rule.

The most important rule that came out of this review was:

**If the commenter validates the poster's feelings but ultimately criticizes what the poster did, the comment is labeled `CRITIQUES_REACTION`.**

## 2. Error Analysis Support

I also used an AI assistant after model evaluation to help identify recurring patterns across misclassified examples.

I directed the tool to look for possible themes such as sarcasm, mixed judgments, short comments, indirect language, class imbalance, and repeated label confusion.

The AI's suggestions were treated as hypotheses rather than conclusions. I manually reread the misclassified examples and kept only patterns that were actually supported by the data. Suggestions that did not occur consistently were discarded.

## 3. Code Debugging

AI assistance was also used to debug the zero-shot baseline implementation, including investigating why Groq responses were not being parsed correctly and checking the relationship between the model output strings and the label mapping.

The final implementation and outputs were verified by rerunning the notebook rather than assuming the AI-generated debugging suggestions were correct.

---

# Limitations

This project has several limitations.

First, the dataset contains only **200 labeled comments**, which is small for learning subtle linguistic distinctions.

Second, the classes are not perfectly balanced. `SUPPORTS_REACTION` represents 60% of the dataset while `CRITIQUES_REACTION` represents 40%.

Third, Reddit comments can depend heavily on context from the original post. A comment may be difficult to interpret when presented independently, even if its judgment was clear within the full discussion.

Fourth, the taxonomy intentionally simplifies nuanced reactions into two mutually exclusive classes. Some Reddit comments contain both support and criticism, requiring an annotation rule that may remove some of the nuance present in the original discussion.

Finally, removing explicit `NOR` and `YOR` verdict tokens reduces lexical leakage but also makes the task harder than ordinary classification of the subreddit.

---

# Conclusion

TakeMeter explores whether a text classifier can distinguish between commenters who support an original poster's reaction and commenters who believe the reaction was excessive or poorly handled.

The fine-tuned DistilBERT model currently achieves **60% accuracy on a 30-example held-out test set**. More important than the aggregate score, however, is whether the model learned the intended conceptual distinction rather than shortcuts based on supportive or critical wording.

The error analysis focuses on that gap between the taxonomy I designed and the decision boundary the model actually learned. Future improvements would center on expanding the dataset, increasing the diversity of `CRITIQUES_REACTION` examples, and deliberately adding more difficult mixed-judgment cases.
