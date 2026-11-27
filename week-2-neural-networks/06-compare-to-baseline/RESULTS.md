# Assignment 06 — Compare to Baseline

## Objective
Compare a Week 1 traditional machine-learning baseline (Logistic Regression), a Week 2 neural network (PyTorch multilayer perceptron), and a third contender (HistGradientBoostingClassifier) on the **same held-out test set**, using identical classification metrics. The task is to predict missed medical appointments (`No-show = Yes`).

## Dataset and experimental setup
- **Dataset:** Medical Appointment No Shows (Kaggle), initially 110,527 rows and 14 columns.
- **Cleaning:** Removed the one record with a negative age, leaving 110,526 records.
- **Target:** `Yes = 1` (missed appointment); `No = 0` (attended).
- **Features:** Excluded patient and appointment identifiers; derived waiting days and scheduled/appointment weekdays from date columns; one-hot encoded gender and neighbourhood. Final feature count: **93**.
- **Class balance:** About **20.19% no-shows** and **79.81% attended**.
- **Split:** 88,420 training-pool records and **22,106 held-out test records** (80/20, stratified, `random_state=42`). For threshold selection, the training pool was further divided into **70,736 fit** and **17,684 validation** records.
- **Fairness:** All three models used the same input features, training/validation partitions, and untouched test set. The validation set—not the test set—was used to select each model's F1-maximizing classification threshold.
- **Models:** Logistic Regression (`max_iter=1000`); HistGradientBoosting (`max_iter=100`, `learning_rate=0.1`, `max_leaf_nodes=31`, `random_state=42`); PyTorch MLP (93 → 64 → 32 → 1, ReLU, Adam at `lr=0.001`, 10 epochs, batch size 256, BCEWithLogitsLoss).

## Why accuracy alone is misleading
With nearly 80% of appointments attended, a classifier predicting **every patient will attend** would achieve about 79.81% accuracy but identify **zero** no-shows. We therefore prioritize **recall and F1-score for the no-show class**, while also reporting precision, accuracy, and false positives.

## Original results at the default 0.50 threshold

| Model | Accuracy | Precision | Recall | F1-score |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.7961 | 0.3854 | 0.0166 | 0.0318 |
| HistGradientBoosting | 0.7992 | 0.5793 | 0.0213 | 0.0411 |
| Neural Network | 0.7982 | 0.5172 | 0.0134 | 0.0262 |

Despite apparently strong accuracy, the default threshold misses almost all no-shows. These were the initial full-training-pool runs, before the separate fit/validation split used below.

## Validation-only threshold selection

| Model | F1-maximizing threshold | Validation F1-score |
|---|---:|---:|
| Logistic Regression | 0.1868 | 0.4061 |
| HistGradientBoosting | 0.2221 | **0.4488** |
| Neural Network | 0.2413 | 0.4456 |

Lower thresholds increase the number of predicted no-shows, improving recall but also increasing false alarms. Thresholds were selected without examining test-set performance.

## Final held-out test results

| Model | Accuracy | Precision (no-show) | Recall (no-show) | F1-score (no-show) | Training time* | Serialized model size |
|---|---:|---:|---:|---:|---:|---:|
| Logistic Regression | 0.6100 | 0.2947 | 0.6682 | 0.4090 | 33.48 s | **3.96 KB** |
| **HistGradientBoosting** | 0.6238 | 0.3194 | **0.7632** | **0.4504** | **10.28 s** | 300.41 KB |
| Neural Network | **0.6519** | **0.3263** | 0.6794 | 0.4408 | 12.51 s | 34.46 KB |

*Training times and sizes were measured in a separate fresh training run on the same 70,736-record fit split, with the same model configurations. The displayed test metrics belong to the preceding fit-split models and validation-selected thresholds; stochastic neural-network runs can differ slightly. Times are single-run measurements, not hardware-independent benchmarks.*

### Performance chart (F1-score, larger is better)

```text
Logistic Regression   0.4090 | █████████████████████████████████████████
HistGradientBoosting  0.4504 | █████████████████████████████████████████████
Neural Network        0.4408 | ████████████████████████████████████████████
```

### Recall chart (fraction of actual no-shows detected)

```text
Logistic Regression   66.82% | █████████████████████████████████
HistGradientBoosting  76.32% | ██████████████████████████████████████
Neural Network        67.94% | ██████████████████████████████████
```

## Confusion matrices

Rows are **actual** `[attended, no-show]`; columns are **predicted** `[attended, no-show]`.

| Model | True negatives | False positives | False negatives | True positives |
|---|---:|---:|---:|---:|
| Logistic Regression | 10,502 | 7,140 | 1,481 | 2,983 |
| HistGradientBoosting | 10,383 | 7,259 | **1,057** | **3,407** |
| Neural Network | 11,379 | **6,263** | 1,431 | 3,033 |

HistGradientBoosting correctly identifies **3,407 of 4,464** missed appointments, but incorrectly flags **7,259** attended appointments. The neural network generates fewer false positives but misses more actual no-shows.

## Estimated beginner development effort

| Model | Estimated learning, coding and debugging time |
|---|---:|
| Logistic Regression | ~2 hours |
| HistGradientBoosting | ~2.5 hours |
| Neural Network | ~5 hours |

**Important:** These are illustrative estimates for a beginner, **not logged or verified personal time measurements**. They should not be represented as actual time spent.

## Winner and interpretation

**Winner: HistGradientBoosting**, when the objective is to identify as many likely no-shows as possible while maintaining a reasonable balance of precision and recall.

- It achieves the **highest test F1-score (0.4504)** and **highest recall (76.32%)**.
- Its measured training run was fastest (**10.28 seconds**), although training times can vary between runs.
- Its serialized size (**300.41 KB**) is larger than the other two models but still small in absolute terms.
- The neural network has the **highest accuracy and precision**, so the winner depends on the cost of false alarms versus missed no-shows.

### Was the neural network worth it?

**Not for this particular tabular dataset and chosen objective.** The neural network required more implementation effort, yet its test F1-score (0.4408) and recall (67.94%) were below HistGradientBoosting. This does not mean neural networks are always inferior: architecture, preprocessing, tuning, and data type can change the outcome.

## Limitations and next steps

1. **False positives remain high.** A real clinic should consider how much unnecessary outreach it can afford and choose a threshold based on operational costs rather than F1 alone.
2. **Single split and single training runs.** Repeated stratified validation and multiple random seeds would provide more reliable comparisons.
3. **Feature and data limitations.** The dataset is historical; real-world deployment would require checks for data drift, leakage, fairness, and feature availability at prediction time.
4. **Development-time estimates are not measurements.** Future assignments should log start/end times for a more defensible comparison.
5. **The model is not deployment-ready.** Further calibration and prospective testing are needed before influencing patient care or outreach decisions.

## Conclusion

The experiment shows why an apparently high accuracy score can be misleading on imbalanced data. Validation-based threshold selection improved no-show recall substantially, but with a large false-positive trade-off. **HistGradientBoosting is the strongest of the three tested models for no-show detection under F1/recall-focused evaluation**, and adding a neural network did not yield a clear benefit for this task.
