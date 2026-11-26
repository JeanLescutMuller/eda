# E. Who will churn, convert or buy?

Scoring each unit (user, customer, account, lead) by its chance of an event in a future window, so that someone can act on the top of the list.

## What you'll hear

- "Which customers are about to churn?"
- "Predict who will convert after the trial."
- "Can we target the users most likely to buy?"
- "Which accounts should sales call first?"

## Ask first

| Ask | Why |
|---|---|
| The label exactly: what event, in which window after which date ("no login for 30 days, within 60 days of the snapshot") | the label's definition is half the model |
| What action follows, and on how many units (call the top 1,000)? | the metric that matters is precision in the top k, not overall accuracy |
| Is the action meant to change the outcome (a retention offer)? | then it's an uplift question, not a propensity one |
| When is the score used, and what's known at that moment? | features must only use what's known at scoring time |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the event is predictable at all | the model's PR-AUC is close to the base rate |
| recent activity drives it (recency, frequency) | a model on those alone is as good as the full one |
| the score is calibrated | the calibration curve departs from the diagonal |
| the top decile holds most of the events | the lift in the top decile is close to 1 |
| targeting the top scores changes outcomes | an uplift test (treated vs held-out within the top) shows no difference |

## The notebooks

1. **Snapshot and label**: pick a snapshot date; features from before it, the label from the window after it. Several snapshots stacked if the history allows (one row per unit per snapshot).
2. **Base rate**: the event's share per snapshot (`px.bar`); it sets the floor for every metric.
3. **Time split**: train on early snapshots, test on the latest. Never a random split across time; grouped by unit when units repeat (`GroupKFold`).
4. **Leakage check**: any feature computed after the snapshot, or a field that's a consequence of the event (a cancellation reason, a closed-account flag); a single feature with a huge importance is a suspect.
5. **Model**: see `E_quick_model.md` (dummy baseline, logistic regression, gradient boosting); `class_weight="balanced"` or nothing, never oversampling the test set.
6. **Ranking**: PR curve and PR-AUC (`sklearn.metrics.precision_recall_curve`, `average_precision_score`), not accuracy; ROC-AUC as a second number.
7. **Lift by decile**: the event rate per score decile (`pd.qcut(score, 10)`), `px.bar(text_auto=".1%")`, the base rate as a line; cumulative gain (share of events captured in the top k %).
8. **Calibration**: `sklearn.calibration.calibration_curve(y, p, n_bins=10)` as `px.line` against the diagonal; recalibrate (`CalibratedClassifierCV`) if the scores are used as probabilities.
9. **Drivers**: permutation importance on the test set, and the event rate per value of the top features (`px.bar`), described as association, not cause.

## The evaluation

On the latest snapshot only: PR-AUC, precision and recall at the top k the action allows, lift in the top decile, calibration. A CI by bootstrap over units (`tests.md`) when comparing two models.

## Traps

| Trap | Fix |
|---|---|
| accuracy on a 3 % event | PR-AUC, precision at k, lift |
| a feature known only after the snapshot | features cut at the snapshot date; check the top features one by one |
| random split across time | time split |
| propensity used for targeting a treatment | uplift: compare treated vs held-out within each score band; the most likely to churn may be the least persuadable |
| a label window that hasn't closed for the latest units | only snapshots whose window has fully passed |
| scores read as probabilities without checking | calibration curve |

## The answer's shape

The score's lift in the top decile (and precision at the action's k), its calibration, the 3–5 features most associated with the event, the base rate it beats, and how the action should be tested (a held-out group).
