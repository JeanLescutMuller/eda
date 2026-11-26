# E. Build a quick model: can we predict X?

A baseline model in about 10 minutes, to answer "is there signal?" and "what drives it?", not to ship. The value is in the honest evaluation and the leakage checks, not in tuning.

## What you'll hear

- "Can you build a quick model to predict whether a task passes?"
- "Is there any signal in these features?"
- "Which features matter most for the outcome?"
- "How well could we predict the price from the rest?"

## Ask first

| Ask | Why |
|---|---|
| The target and when it's known; which columns exist before it | anything known only after the target is leakage |
| The unit and whether it repeats (several rows per user, item, run) | the split must keep each unit on one side |
| What "good enough" means: beating a rule, a current model, a threshold | the baseline to beat |
| Is it for prediction, or for explaining what drives the target? | prediction: test score; explanation: importance and partial dependence, read as association |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| there's signal | the model doesn't beat the dummy baseline by more than the CV spread |
| a simple model is enough | gradient boosting doesn't beat logistic / linear regression |
| feature F drives the target | permuting F barely changes the test score |
| no leakage | no single feature gives a near-perfect score, and the score holds on a later period |

## The notebooks

1. **Target and features**: the target's distribution (`px.histogram`), the base rate or its mean; drop ids, free text and any column produced after the target.
2. **Pipeline**: one `sklearn.compose.ColumnTransformer` (`OneHotEncoder(handle_unknown="ignore")` for categories, `StandardScaler` or passthrough for numbers, `SimpleImputer`), inside a `sklearn.pipeline.make_pipeline` so the preprocessing is fit only on the training folds.
3. **Baseline**: `DummyClassifier(strategy="prior")` / `DummyRegressor()`; its score is the floor.
4. **Two models**: `LogisticRegression(max_iter=1000)` (or `Ridge`) and `HistGradientBoostingClassifier` (or `Regressor`): handles missing values and needs little tuning.
5. **Cross-validation**: `cross_validate(pipe, X, y, cv=GroupKFold(5), groups=df.unit, scoring=[...])`; `TimeSeriesSplit` if time matters. Plot the score per fold and model (`px.strip`), the dummy as a line.
6. **Metric**: classification: ROC-AUC and PR-AUC (imbalanced), log loss; regression: MAE and R², compared to the dummy.
7. **Importance**: `sklearn.inspection.permutation_importance` on held-out data, as a sorted `px.bar` with its std as `error_x`; not the model's built-in importance (biased toward many-valued features).
8. **Leakage check**: one feature far above the rest, or a near-perfect score: drop it, rerun, and explain what it is.
9. **Direction**: for the top 3 features, `sklearn.inspection.partial_dependence` or the target's mean per bin (`px.line`).

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
