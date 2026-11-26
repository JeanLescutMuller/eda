# E. Build a quick model: can we predict X?

A baseline model in about 10 minutes, to answer "is there signal?" and "what drives it?", not to ship. The value is in the honest evaluation and the leakage checks, not in tuning.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| there's signal | the model doesn't beat the dummy baseline by more than the CV spread |
| a simple model is enough | gradient boosting doesn't beat logistic / linear regression |
| feature F drives the target | permuting F barely changes the test score |
| no leakage | no single feature gives a near-perfect score, and the score holds on a later period |

## The evaluation

Cross-validated scores, mean ± std across folds, against the dummy; two models compared fold by fold (paired, `tests.md`) only if the human asks which is better.

## Traps

| Trap | Fix |
|---|---|
| rows of the same unit in train and test | `GroupKFold` |
| preprocessing fit on all the data | everything inside the pipeline |
| a feature computed after the target | drop it; check the top feature first |
| accuracy on an imbalanced target | ROC-AUC, PR-AUC |
| importance read as cause | "associated with", plus a direction plot |
| hours spent tuning | default hyperparameters; the question is signal, not the last point |

## The answer's shape

Signal or not: the best model's cross-validated score against the dummy (mean ± std), the 3–5 features that carry it with their direction, the leakage checks done, and what a real model would need next.
