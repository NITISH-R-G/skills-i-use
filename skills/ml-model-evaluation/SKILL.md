---
name: ml-model-evaluation
description: Reference for evaluating machine learning models correctly — picking the right metric for the problem, detecting overfitting, and designing a validation strategy that gives an honest estimate of real-world performance. Use this whenever the user is choosing evaluation metrics for a model, debugging why a model performs well in testing but poorly in production, setting up train/validation/test splits, or reviewing model evaluation results for validity.
---

# ML Model Evaluation

The central risk in model evaluation isn't picking a bad model — it's picking a *good-looking but wrong evaluation*, which then confidently recommends a model that will disappoint in production. Every practice here exists to close a specific way evaluation numbers can lie.

## Picking the right metric — accuracy is usually the wrong default

**Why raw accuracy misleads on imbalanced data**: a classifier that always predicts "not fraud" on a dataset where 99% of transactions are legitimate scores 99% accuracy while catching zero actual fraud — the single most common metric-selection mistake. Before reaching for accuracy, check the class balance; if it's meaningfully skewed, accuracy is close to meaningless as a standalone metric.

**Precision vs. recall — the trade-off is the real decision, not a formula**:
- **Precision**: of everything flagged positive, what fraction actually was? High precision means few false alarms.
- **Recall**: of everything actually positive, what fraction did the model catch? High recall means few misses.

These trade off against each other — a model can be tuned toward one at the cost of the other by moving its decision threshold. **Which one matters more is a business/domain question, not a modeling question**: for spam detection, a false positive (a real email marked spam) is often more costly to the user than a false negative (a spam email that gets through) — favor precision. For cancer screening, a false negative (missing an actual case) is far more costly than a false positive (an unnecessary follow-up test) — favor recall. Choosing the metric to optimize *before* modeling starts, based on which error type costs more in context, keeps the modeling effort pointed at the actual goal instead of an arbitrary default.

**F1 score** (the harmonic mean of precision and recall) is a reasonable single number when both matter roughly equally and you want one metric to optimize or compare across model versions — but it obscures *which* of precision/recall is driving a change, so it's better as a summary tracked alongside the two components, not a replacement for looking at both.

**ROC-AUC vs. precision-recall AUC**: ROC-AUC is the standard choice for balanced classes; on heavily imbalanced data, precision-recall AUC is more informative because ROC-AUC can look deceptively good even for a weak model, since the true-negative rate (which dominates ROC-AUC's denominator on imbalanced data) is easy to get right when negatives vastly outnumber positives.

**For regression**: MAE (mean absolute error) is more robust to outliers and easier to interpret directly in the target's units; RMSE (root mean squared error) penalizes large errors more heavily, which is the right choice specifically when large errors are disproportionately costly in the domain (a large miss matters much more than several small ones) — pick based on whether that's true for the problem at hand, not by default.

**The general principle**: the metric should be chosen to reflect what actually matters for the decision the model informs, before looking at any results — picking a metric *after* seeing which one makes the model look best is optimizing the report, not the model, and it's a subtle form of the same problem A/B testing's "decide the metric before the test runs" principle exists to prevent (see `requirements-engineering`).

## Overfitting — the model memorized instead of learned

**What it is**: a model that performs well on the data it was trained on but poorly on new, unseen data — it captured noise and idiosyncrasies specific to the training set rather than the general pattern that would transfer.

**How to detect it**: a large, persistent gap between training performance and validation performance is the direct signal — a model scoring 98% on training data and 71% on validation data has memorized rather than generalized. A small, stable gap is normal and expected; a large or growing gap is the warning sign.

**Common causes and matching fixes**:
- **Model too complex for the amount of data available** — reduce model capacity (fewer parameters, shallower trees, simpler architecture), or get more training data if the complexity is genuinely needed for the problem.
- **Training for too long** (particularly for iteratively-trained models like neural networks) — the model fits noise as an increasing fraction of remaining unfit variance is noise rather than signal. Early stopping (halt training once validation performance stops improving, even if training performance keeps climbing) directly targets this.
- **Too few examples relative to the pattern's real complexity** — regularization (penalizing model complexity directly during training) forces the model toward simpler explanations that are more likely to generalize, at some cost to how tightly it fits the training data.
- **Data leakage** — information from outside the legitimate training set (future data, or a feature that's a proxy for the target that wouldn't be available at prediction time) sneaking into training, producing validation performance that looks great but reflects information the model won't have in production. This is the most dangerous cause because it can survive even careful cross-validation if the leak is systemic to how the dataset was constructed — worth actively auditing for, not just assuming cross-validation would have caught it.

**Underfitting, the opposite failure**: poor performance on *both* training and validation data — the model isn't complex enough to capture the real pattern, or hasn't trained long enough. The fix is close to the mirror image of overfitting's fixes: more capacity, more relevant features, longer training, less aggressive regularization. Checking both training and validation performance together, not validation alone, is what distinguishes "underfit" from "overfit" from "well-fit" — validation performance alone can't tell those apart.

## Validation strategy — getting an honest estimate

**The basic three-way split, and why each part exists**:
- **Training set** — what the model learns from.
- **Validation set** — used *during* development to tune hyperparameters and make modeling decisions (which architecture, which regularization strength). Because it influences decisions, performance on it is somewhat optimistic — the modeling process has implicitly been tuned to do well on this specific data, even without directly training on it.
- **Test set** — touched exactly once, at the very end, after all modeling decisions are finalized, to produce the actual unbiased estimate of real-world performance.

**Why the test set has to stay untouched until the end**: every time a modeling decision is made by checking performance on a specific dataset, that dataset's ability to give an honest estimate degrades a little — this is why the validation set (checked repeatedly during development) isn't a reliable final estimate, and why a fresh test set, checked exactly once, is needed to get a number that hasn't been implicitly tuned toward. Checking test-set performance repeatedly during development and adjusting based on it defeats the entire purpose of having a separate test set — at that point it's functioning as a second validation set, not an honest final check.

**Cross-validation** (k-fold: split the data into k parts, train on k-1, validate on the remaining one, rotate through all k combinations, average the results): gives a more robust performance estimate than a single train/validation split, especially valuable when the dataset is small enough that a single split's performance could be heavily influenced by which specific examples happened to land in which set. The cost is k times the compute for k times the confidence — worth it more often for smaller datasets where a single split's luck-of-the-draw risk is highest.

**Time-based splits, when data has a temporal dimension.** If the model will be used to predict the future from data up to "now," a random train/validation split leaks future information backward into training in a way that inflates validation performance unrealistically — a model that's seen randomly-scattered examples from after the point it's supposed to be predicting from has an advantage it won't have in real deployment. Split chronologically instead: train on data up to a point in time, validate on data after that point, mirroring how the model will actually be used.

**Watch for the same leakage risk in cross-validation groupings that matters for the domain** — if multiple rows in the dataset come from the same underlying entity (multiple transactions from one customer, multiple images of one patient), a random split can put some of that entity's rows in training and others in validation, letting the model implicitly learn entity-specific patterns that inflate validation performance without reflecting how well it'll generalize to a genuinely new entity. Group-aware splitting (keeping all of one entity's rows together in the same fold) avoids this specific leak.

## Practical checklist

- Was the evaluation metric chosen based on the actual cost of different error types in this domain, decided before looking at results?
- Is there a meaningful gap between training and validation performance? If so, which direction (over- vs. under-fit), and does the fix match?
- Has the test set been touched more than once? If so, its performance estimate is no longer trustworthy as an honest final number.
- Does the data have a temporal or grouped structure that a naive random split would leak across?
- If the classes/target are imbalanced, is the reported metric one that's actually informative under that imbalance (not raw accuracy, and consider precision-recall AUC over ROC-AUC)?
