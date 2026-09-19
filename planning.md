# TakeMeter Planning

## Community

I chose **r/AmIOverreacting** because the community is based on judging whether someone’s reaction was reasonable or too much.

I will use **comments**, not the original posts. The comments are useful because people clearly support, criticize, or question the original poster’s reaction. The topics also vary across relationships, family, friendships, work, money, and personal boundaries.

## Labels

### SUPPORTS_REACTION

The commenter believes the original poster’s reaction or feelings were reasonable.

Examples:

* “You already told him that bothered you. You have every right to be upset.”
* “You are not wrong for feeling hurt by what happened.”

### CRITIQUES_REACTION

The commenter believes the original poster overreacted, handled the situation poorly, or responded too strongly.

Examples:

* “Being upset makes sense, but ending the friendship over this is too much.”
* “You are assuming the worst without even talking to them first.”

## Hard Edge Cases

The hardest comments will be ones that agree with the person’s feelings but disagree with how they reacted.

Example:

> “You have every right to be angry, but breaking up over this feels extreme.”

I would label this `CRITIQUES_REACTION` because the commenter believes the actual reaction went too far.

I will leave out comments that are only jokes, questions, unrelated conversations, or advice with no clear judgment.

## Data Collection Plan

I will collect at least **200 top-level comments** from r/AmIOverreacting.

My goal is around:

* 100 `SUPPORTS_REACTION`
* 100 `CRITIQUES_REACTION`

Before labeling all 200, I will first review around **30–40 comments** to make sure the labels work well.

If one label is underrepresented, I will collect more comments from different threads instead of forcing comments into a label.

## Evaluation Metrics

I will use:

* **Accuracy** to measure overall performance.
* **Precision** to see how often each predicted label is correct.
* **Recall** to see how many examples from each label the model correctly finds.
* **F1 score** to balance precision and recall.
* **Confusion matrix** to see which labels the model mixes up.

I will also compare the fine-tuned model to the zero-shot baseline on the same test set.

## Definition of Success

I would consider the model successful if it gets:

* at least **80% accuracy**
* at least **0.75 F1 score for both labels**
* better performance than the zero-shot baseline

For a real community tool, I would want around **85% accuracy and 0.80 F1** for both labels before trusting it as an assistive tool.

## AI Tool Plan

### Label Stress-Testing

I will ask an AI tool to create 5–10 comments that are difficult to classify between my two labels.

If I cannot label them clearly, I will improve my label definitions before labeling the full dataset.

### Annotation Assistance

I may use an LLM to pre-label some comments, but I will review every label myself before adding it to the final dataset.

I will keep track of which examples were AI-assisted.

### Failure Analysis

After testing the model, I will give the incorrect predictions to an AI tool and ask it to find patterns.

I will look for issues like:

* sarcasm
* very short comments
* mixed opinions
* indirect language
* comments that need context from the original post

I will check those patterns myself before including them in my final report.

## Main Risk

One risk is that the model learns simple words like **“NO”** or **“YES”** instead of understanding the full comment.

I will check whether the model still performs well when those obvious words are not present.

Another risk is class imbalance, so I will collect comments from many different posts instead of taking too many from one thread.
