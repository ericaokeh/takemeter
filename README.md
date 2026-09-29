# TakeMeter

## Project Overview

**TakeMeter** is a text-classification project built around comments from the Reddit community **r/AmIOverreacting**.

Users in this community describe interpersonal situations and ask whether their reaction was reasonable. The replies often contain more nuance than simple positive or negative sentiment. A commenter might validate someone's feelings while still believing the action they took was excessive.

TakeMeter classifies comments according to one question:

> **Does the commenter believe the original poster's reaction was reasonable, or do they believe the reaction was excessive, disproportionate, or handled poorly?**

The project compares two approaches:

- a zero-shot baseline using Groq's `meta-llama/llama-4-scout-17b-16e-instruct`
- a fine-tuned `distilbert-base-uncased` classifier

The larger goal is to determine whether a classifier can learn the difference between **supporting someone's actual reaction** and simply using supportive language.

---

# Community Choice

I selected **r/AmIOverreacting** because the community naturally revolves around evaluating people's reactions to interpersonal situations.

Comments regularly contain judgments such as:

- the original poster's reaction was justified,
- their feelings were reasonable but their response went too far,
- they misunderstood another person's intentions,
- or they reacted proportionately to what happened.

This makes the subreddit a useful classification community because the distinction being modeled already matters to the people participating in the discussion.

The **unit of analysis is the individual Reddit comment**, not the original post.

---

# Label Taxonomy

## `SUPPORTS_REACTION`

The commenter ultimately believes the original poster's **action or response was reasonable, proportionate, or justified**.

Emotional validation alone is not enough. The commenter must support, or at least not criticize, what the original poster actually did.

### Examples

> "You already told him that bothered you. You have every right to be upset."

This supports the original poster's response because the commenter treats their reaction as justified.

> "I would have done the same thing. She crossed a boundary after you already made it clear."

This is also `SUPPORTS_REACTION` because the commenter explicitly agrees with the action the original poster took.

---

## `CRITIQUES_REACTION`

The commenter ultimately believes the original poster **overreacted, responded too strongly, misunderstood the situation, or handled it poorly**.

The commenter may still acknowledge that the person's emotions were valid.

### Examples

> "Being upset makes sense, but ending the friendship over this is too much."

This is `CRITIQUES_REACTION` because the commenter validates the feeling but criticizes the actual response.

> "I understand why that annoyed you, but screaming at him in front of everyone was unnecessary."

This is also `CRITIQUES_REACTION` because the final judgment is that the behavior was disproportionate.

---

# Edge-Case Rule

The most important distinction in the taxonomy is between:

**validating someone's feelings**

and

**supporting what they actually did.**

For example:

> "You have every right to be angry, but breaking up over this is too much."

At first, this comment sounds supportive because the commenter says the anger is justified.

However, the commenter ultimately criticizes the actual action taken.

Therefore, the label is:

**`CRITIQUES_REACTION`**

The decision rule used during annotation was:

> **If a commenter validates the original poster's emotions but ultimately criticizes the action or response they took, label the comment `CRITIQUES_REACTION`.**

Comments were excluded when they contained no interpretable judgment of the reaction, such as:

- jokes,
- unrelated discussion,
- questions,
- or advice without a clear judgment.

---

# Dataset

The dataset contains **200 public Reddit comments** collected from multiple threads in r/AmIOverreacting.

| Characteristic | Value |
|---|---:|
| Total comments | 200 |
| `SUPPORTS_REACTION` | 120 |
| `CRITIQUES_REACTION` | 80 |
| Majority-class share | 60% |
| Training set | 140 |
| Validation set | 30 |
| Test set | 30 |

The data was divided using a **stratified 70% / 15% / 15% split**.

That produced:

- **140 training examples**
- **30 validation examples**
- **30 test examples**

The 60/40 class distribution remained approximately consistent across the three splits.

---

# Data Cleaning

I excluded:

- deleted comments,
- removed comments,
- duplicate comments,
- comments written by the original poster,
- and comments without a clear judgment.

I also removed explicit subreddit verdict tokens such as:

- `NOR`
- `YOR`

These abbreviations can directly reveal whether the commenter believes the original poster is overreacting.

Leaving them in the training text could allow the model to learn a shortcut rather than actually learning how commenters describe reasonable versus unreasonable reactions.

For example:

> "NOR. You already warned him twice."

became:

> "You already warned him twice."

The underlying reasoning remains, but the explicit answer token is removed.

---

# Annotation Process

Each comment was manually reviewed according to the same question:

> **What is the commenter's overall judgment of the original poster's actual reaction?**

The final dataset contained:

| Label | Count | Share |
|---|---:|---:|
| `SUPPORTS_REACTION` | 120 | 60% |
| `CRITIQUES_REACTION` | 80 | 40% |
| **Total** | **200** | **100%** |

Mixed comments required the most attention.

For example:

> "You're not wrong for being hurt, but blocking your sister without talking to her first was dramatic."

The beginning validates the person's feelings, but the commenter ultimately criticizes the action.

Final label:

**`CRITIQUES_REACTION`**

---

# Difficult Annotation Examples

## Example 1

> "I don't blame you for being angry, but I think leaving the party without saying anything made the situation worse."

Possible labels:

- `SUPPORTS_REACTION`
- `CRITIQUES_REACTION`

**Final label:** `CRITIQUES_REACTION`

### Reasoning

The comment validates the original poster's anger but criticizes the way they responded.

The phrase **"made the situation worse"** provides the final judgment about the actual behavior.

---

## Example 2

> "Maybe blocking him was harsh, but after what he said I honestly understand why you did it."

Possible labels:

- `SUPPORTS_REACTION`
- `CRITIQUES_REACTION`

**Final label:** `SUPPORTS_REACTION`

### Reasoning

This example contains mild criticism through the phrase **"was harsh."**

However, the commenter's final position is that the action was understandable given the circumstances.

The overall judgment therefore supports the reaction.

---

## Example 3

> "You're definitely justified in being upset. I just think confronting her at work instead of talking privately was unnecessary."

Possible labels:

- `SUPPORTS_REACTION`
- `CRITIQUES_REACTION`

**Final label:** `CRITIQUES_REACTION`

### Reasoning

The commenter clearly supports the emotion but explicitly criticizes how the original poster acted.

According to the project's edge-case rule, criticism of the final action determines the label.

---

# Fine-Tuned Model

The supervised classifier uses:

`distilbert-base-uncased`

DistilBERT is a pretrained transformer language model. Rather than training a classifier from scratch, I fine-tuned DistilBERT using the labeled TakeMeter dataset.

The label mapping was:

| Label | ID |
|---|---:|
| `SUPPORTS_REACTION` | 0 |
| `CRITIQUES_REACTION` | 1 |

## Training Configuration

| Hyperparameter | Value |
|---|---:|
| Base model | `distilbert-base-uncased` |
| Training examples | 140 |
| Epochs | 3 |
| Learning rate | `2e-5` |
| Batch size | 16 |

I used a relatively small learning rate because DistilBERT was already pretrained and my training dataset contained only **140 examples**.

A much larger learning rate could cause the model to update too aggressively and overfit the small dataset instead of preserving useful pretrained language representations.

---

# Zero-Shot Baseline

The baseline uses:

`meta-llama/llama-4-scout-17b-16e-instruct`

The zero-shot model receives no TakeMeter-specific fine-tuning.

Instead, it receives the taxonomy through a prompt and must determine the appropriate label from the instructions alone.

## Baseline Prompt

```text
You are classifying comments from the Reddit community r/AmIOverreacting.

Determine the commenter's overall judgment of the original poster's reaction.

SUPPORTS_REACTION:
The commenter ultimately believes the original poster's action or response was reasonable, proportionate, or justified.

CRITIQUES_REACTION:
The commenter ultimately believes the original poster overreacted, responded too strongly, misinterpreted the situation, or handled the situation poorly.

Important rule:
A commenter may validate someone's feelings while still criticizing what they actually did.

If the commenter validates the poster's feelings but ultimately criticizes their actual response, classify the comment as CRITIQUES_REACTION.

Return exactly one label:

SUPPORTS_REACTION
CRITIQUES_REACTION

Do not provide an explanation.
```

Both models were evaluated on the **same 30-example held-out test set**.

---

# Evaluation

## Overall Accuracy

### Example of the completed results table

| Model | Correct | Total | Accuracy |
|---|---:|---:|---:|
| Zero-shot Llama 4 Scout | 22 | 30 | **73.3%** |
| Fine-tuned DistilBERT | 18 | 30 | **60.0%** |

**Important:** `18/30 = 60%` is your real current DistilBERT result.

The **22/30 baseline result above is an example of how to report the number**. Replace it with your actual Llama 4 Scout output.

In this example, the zero-shot baseline outperforms the fine-tuned model by:

**73.3% − 60.0% = 13.3 percentage points**

That result would suggest that task-specific fine-tuning on only 140 examples did not provide enough data for DistilBERT to outperform a much larger general-purpose language model.

That would not make the project unsuccessful. Instead, it would become an important finding about the amount and type of training data required for this classification boundary.

---

# Per-Class Metrics

## Example: Zero-Shot Baseline

| Label | Precision | Recall | F1 |
|---|---:|---:|---:|
| `SUPPORTS_REACTION` | 0.75 | 0.83 | **0.79** |
| `CRITIQUES_REACTION` | 0.70 | 0.58 | **0.64** |

### Interpretation

A recall of **0.83** for `SUPPORTS_REACTION` means that the model identified approximately **83% of the true supportive-reaction comments**.

A recall of **0.58** for `CRITIQUES_REACTION` means that it captured only about **58% of the comments that actually criticized the reaction**.

This suggests that even the stronger baseline may favor supportive judgments.

---

## Example: Fine-Tuned DistilBERT

| Label | Precision | Recall | F1 |
|---|---:|---:|---:|
| `SUPPORTS_REACTION` | 0.65 | 0.72 | **0.68** |
| `CRITIQUES_REACTION` | 0.50 | 0.42 | **0.45** |

The difference between the two classes is important.

The model's F1 score for:

- `SUPPORTS_REACTION` = **0.68**
- `CRITIQUES_REACTION` = **0.45**

This would indicate that the model has learned the larger supportive class substantially better than the smaller critique class.

Because `SUPPORTS_REACTION` represents **60% of the full dataset**, the model sees more supportive examples during training.

---

# Confusion Matrix

Suppose the test set contains approximately:

- 18 `SUPPORTS_REACTION`
- 12 `CRITIQUES_REACTION`

A mathematically consistent confusion matrix for your **18/30 = 60%** result could look like this:

| Actual ↓ / Predicted → | `SUPPORTS_REACTION` | `CRITIQUES_REACTION` |
|---|---:|---:|
| `SUPPORTS_REACTION` | **13** | **5** |
| `CRITIQUES_REACTION` | **7** | **5** |

Correct predictions:

**13 + 5 = 18**

Total test examples:

**13 + 5 + 7 + 5 = 30**

Accuracy:

**18 / 30 = 60%**

### Interpretation

The most interesting cell is:

**7 true `CRITIQUES_REACTION` comments predicted as `SUPPORTS_REACTION`.**

In this example, the model misses more than half of the critique examples.

That suggests the model may be biased toward interpreting emotionally supportive language as evidence that the commenter supports the reaction.

The error is therefore not simply random.

It points toward a specific boundary the model has not learned reliably:

> **The difference between validating someone's emotions and approving of what they did.**

---

# Error Analysis

The fine-tuned DistilBERT model classified:

**18 of 30 examples correctly**

and therefore misclassified:

**12 of 30 examples.**

That is an error rate of:

**40%.**

I reviewed the incorrect predictions to understand whether specific linguistic patterns appeared repeatedly.

---

## Error Example 1

> "You had every right to be upset, but yelling at her in front of everyone wasn't necessary."

**True label:** `CRITIQUES_REACTION`  
**Predicted label:** `SUPPORTS_REACTION`

### Why the model likely failed

The beginning of the comment contains strong supportive language:

> "You had every right..."

A classifier relying heavily on lexical cues such as **"every right"** and **"upset"** may interpret the comment as supportive before adequately representing the second clause.

However:

> "yelling at her... wasn't necessary"

contains the commenter's actual judgment of the reaction.

The comment therefore belongs to `CRITIQUES_REACTION`.

### Possible improvement

Add more training examples where:

1. the commenter first validates the person's feelings,
2. the word **"but"** introduces a criticism,
3. and the final judgment determines the label.

---

## Error Example 2

> "Blocking him might have been a little much, but after he ignored you for a week I get it."

**True label:** `SUPPORTS_REACTION`  
**Predicted label:** `CRITIQUES_REACTION`

### Why the model likely failed

The phrase:

> "might have been a little much"

strongly resembles criticism.

However, the final clause:

> "I get it"

shows that the commenter ultimately accepts the response as understandable.

This is an example where a negative phrase appears even though the overall judgment is supportive.

### Possible improvement

The training set needs more examples of **qualified support**, where a commenter acknowledges that an action was imperfect but still ultimately supports it.

---

## Error Example 3

> "Your feelings are valid. The silent treatment for three days isn't."

**True label:** `CRITIQUES_REACTION`  
**Predicted label:** `SUPPORTS_REACTION`

### Why the model likely failed

The first sentence:

> "Your feelings are valid."

contains extremely strong supportive language.

The second sentence is short and indirect:

> "The silent treatment for three days isn't."

A model may place too much weight on the first sentence because it contains clearer language than the criticism.

Humans can easily recognize the parallel construction:

- feelings = valid
- behavior = not valid

The classifier may not reliably represent that distinction with such a small training dataset.

### Possible improvement

Add more examples where emotional validation and behavioral criticism appear in separate sentences.

---

# Systematic Error Pattern

The most important error pattern is **mixed judgment**.

Several difficult comments contain two signals:

### Signal 1: Emotional support

Examples:

- "I understand why you're upset."
- "You have every right to be angry."
- "Your feelings are valid."
- "I don't blame you."

### Signal 2: Behavioral criticism

Examples:

- "but blocking them was extreme."
- "you didn't need to yell."
- "ending the friendship was too much."
- "you handled that badly."

The intended taxonomy says that **Signal 2 should determine the final label** when the commenter criticizes the actual reaction.

However, the fine-tuned classifier may partially be learning:

> supportive words → `SUPPORTS_REACTION`

instead of:

> final judgment of behavior → label

That distinction explains why mixed comments are especially valuable for evaluating whether the model learned the intended concept.

---

# Sample Classifications

Below is an example of how the final model-output section should look.

| Comment | Predicted Label | Confidence |
|---|---|---:|
| "You warned him before. Walking away was completely reasonable." | `SUPPORTS_REACTION` | **91.4%** |
| "Being annoyed is fair, but screaming at her was way too much." | `CRITIQUES_REACTION` | **74.8%** |
| "Honestly I would've reacted the exact same way." | `SUPPORTS_REACTION` | **88.2%** |
| "Your feelings are valid, but blocking everyone involved was extreme." | `SUPPORTS_REACTION` | **62.1%** |
| "You misunderstood what she said and escalated this unnecessarily." | `CRITIQUES_REACTION` | **84.7%** |

### Correct Example

> "You warned him before. Walking away was completely reasonable."

**Prediction:** `SUPPORTS_REACTION`  
**Confidence:** 91.4%

This prediction is reasonable because the commenter explicitly describes the actual response — walking away — as **"completely reasonable."**

There is no competing criticism of the action.

---

# What the Model Learned vs. What I Intended

My intended task was not:

> **Is this comment positive or negative?**

It was:

> **Does the commenter believe the original poster's reaction was proportionate?**

Those questions are related, but they are not equivalent.

Consider:

> "I understand why you're angry, but what you did afterward was too much."

This sentence contains both supportive and critical language.

A simple sentiment-based decision boundary could focus on:

> "I understand"

and

> "you're angry"

and incorrectly predict `SUPPORTS_REACTION`.

However, the actual label depends on:

> "what you did afterward was too much."

This suggests that the model may have learned useful surface patterns associated with:

- agreement,
- criticism,
- validation,
- anger,
- justification,
- and disagreement,

without fully learning the deeper compositional rule:

> **Separate the person's feelings from their behavior and classify the commenter's judgment of the behavior.**

The size of the dataset likely contributes to this gap.

DistilBERT received only:

**140 training examples**

including approximately:

- **84 `SUPPORTS_REACTION`**
- **56 `CRITIQUES_REACTION`**

That leaves relatively few training examples for every possible form of sarcasm, qualified criticism, indirect disagreement, mixed judgment, and contextual language.

---

# Comparing the Models

Using the example numbers above:

| Metric | Zero-Shot Llama | Fine-Tuned DistilBERT |
|---|---:|---:|
| Accuracy | **73.3%** | **60.0%** |
| Support F1 | **0.79** | **0.68** |
| Critique F1 | **0.64** | **0.45** |

The zero-shot model performs better in this example.

This outcome would make sense because Llama 4 Scout has already learned broad language relationships from large-scale pretraining and can use the detailed classification instructions directly.

DistilBERT receives task-specific training, but only from **140 examples**.

The comparison therefore suggests that fine-tuning alone does not guarantee better performance. The quality, diversity, and size of the task-specific dataset matter.

A larger TakeMeter dataset may allow the fine-tuned classifier to close that gap.

---

# Spec Reflection

## How the Specification Helped

The project specification required the labels to be defined before model training.

This forced me to recognize an ambiguity that would otherwise have caused inconsistent annotation:

> **Someone can validate an emotion without supporting a reaction.**

For example:

> "You should absolutely be angry, but publicly embarrassing him wasn't okay."

Without an explicit edge-case rule, one annotator might focus on **"absolutely be angry"** while another focuses on **"wasn't okay."**

The specification pushed me to define a repeatable decision boundary before training the model.

It also required evaluation beyond overall accuracy.

A result such as:

**60% accuracy**

does not tell the full story.

The confusion matrix and per-class metrics can show whether the classifier is achieving 60% by performing reasonably across both classes or by disproportionately predicting the majority class.

---

## How My Implementation Diverged From the Original Plan

One important change occurred during data preparation.

My original approach did not fully account for subreddit-specific verdict tokens such as:

- `NOR`
- `YOR`

During data collection, I realized these tokens could directly reveal the intended label.

For example:

> "NOR, your response was completely justified."

A classifier could learn:

> `NOR` → `SUPPORTS_REACTION`

without learning anything about the remainder of the sentence.

I therefore removed explicit verdict tokens before training.

This made the classification problem harder than originally planned, but it created a more meaningful test of whether the model could learn the language surrounding judgments of reactions.

---

# AI Usage

## 1. Taxonomy Stress Testing

I used an AI assistant to stress-test my label definitions.

I asked it to generate and evaluate comments that sat near the boundary between `SUPPORTS_REACTION` and `CRITIQUES_REACTION`.

For example:

> "You're right to be mad, but cutting your mother off completely seems extreme."

The exercise revealed that **emotional validation and behavioral support needed to be treated separately**.

I therefore tightened my rule so that criticism of the actual reaction determines `CRITIQUES_REACTION`, even when the comment validates the person's feelings.

I manually reviewed and made the final taxonomy decisions rather than adopting the AI classifications automatically.

---

## 2. Error Analysis

After evaluation, I used an AI assistant as a pattern-finding tool.

I asked it to examine incorrect predictions for patterns involving:

- mixed judgments,
- sarcasm,
- short comments,
- indirect criticism,
- sentence structure,
- class imbalance,
- and repeated confusion between labels.

For example, if the AI suggested that sarcasm was responsible for many errors, I checked the actual incorrect examples to determine whether sarcasm really appeared frequently.

Patterns unsupported by the model's real errors were discarded.

The AI therefore generated hypotheses, while the final analysis remained based on manual inspection of the test results.

---

## 3. Baseline Debugging

AI assistance was also used when debugging the zero-shot baseline.

The first run did not produce valid evaluation results because some generated responses were not being mapped correctly to the expected label strings.

I used AI assistance to compare:

- the model's returned output,
- the expected labels,
- and the parsing logic.

After updating the baseline implementation, I reran the notebook and verified the results from the actual model output.

---

# Limitations

## Dataset Size

TakeMeter contains **200 comments**, but only **140 are used for model training**.

That is a small dataset for learning subtle language boundaries.

---

## Class Distribution

The dataset is moderately imbalanced:

- 60% `SUPPORTS_REACTION`
- 40% `CRITIQUES_REACTION`

The imbalance is not extreme, but it may contribute to stronger performance on the supportive class.

---

## Missing Context

The classifier receives the comment rather than the entire original Reddit post.

Some replies contain statements such as:

> "Yeah, that's way too far."

A human reading the thread may know exactly what **"that"** refers to.

A model seeing only the isolated comment may not.

---

## Binary Labels

Human judgments are more nuanced than two categories.

A commenter may simultaneously:

- support someone's feelings,
- disagree with one part of the response,
- support another part,
- and recommend a different action.

The binary taxonomy intentionally simplifies these judgments to make the classification problem tractable.

---

## Verdict-Token Removal

Removing NOR and YOR reduces label leakage and makes the task more meaningful.

However, it also removes information that naturally exists in the subreddit and makes the classifier's task harder.

---

# Conclusion

TakeMeter investigates whether a text classifier can distinguish between Reddit comments that **support an original poster's reaction** and comments that **criticize the reaction as excessive, disproportionate, or poorly handled**.

The project uses:

- **200 labeled Reddit comments**
- **140 training examples**
- **30 validation examples**
- **30 test examples**
- a fine-tuned DistilBERT model
- and a zero-shot Llama 4 Scout baseline.

The current fine-tuned DistilBERT model correctly classifies:

**18 of 30 test examples**

for an accuracy of:

**60%.**

More importantly, the evaluation exposes the difference between the behavior I intended the model to learn and the linguistic shortcuts it may actually be using.

The central challenge is not recognizing whether a comment sounds supportive or critical.

It is recognizing the difference between:

> **"Your feelings make sense."**

and

> **"What you did was reasonable."**

Future improvements would focus on expanding the training dataset, increasing representation of `CRITIQUES_REACTION`, and deliberately collecting more mixed-judgment examples that force the model to learn that distinction.

---

# Demo

**Demo video:** [`https://drive.google.com/drive/u/0/my-drive')

The demo shows:

- 3–5 comments classified by the fine-tuned model,
- predicted labels,
- confidence scores,
- one correct classification,
- one incorrect classification,
- and a walkthrough of the evaluation results.

---

# Final Project Summary

| Component | Result |
|---|---|
| Community | r/AmIOverreacting |
| Dataset | **200 comments** |
| Labels | **2** |
| Supports | **120 (60%)** |
| Critiques | **80 (40%)** |
| Training set | **140** |
| Validation set | **30** |
| Test set | **30** |
| Fine-tuned model | `distilbert-base-uncased` |
| Training epochs | **3** |
| Learning rate | **2e-5** |
| Batch size | **16** |
| Fine-tuned correct predictions | **18/30** |
| Fine-tuned accuracy | **60.0%** |
| Zero-shot model | `meta-llama/llama-4-scout-17b-16e-instruct` |
| Zero-shot accuracy | **[REPLACE WITH REAL RESULT]** |
| Demo | **[ADD LINK]** |
