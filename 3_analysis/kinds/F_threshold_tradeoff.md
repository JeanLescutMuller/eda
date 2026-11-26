# F. Where should the cut-off be?

Choosing a threshold on a score (fraud, spam, a quality filter, an alert, a model's confidence) so that the errors on each side cost the least, given what each kind of error costs and how often each class occurs.

## What you'll hear

- "At what score should we block a transaction?"
- "Our filter removes too much good data; where should the cut-off be?"
- "We get too many alerts; how do we cut them without missing incidents?"
- "What confidence should the model need before answering on its own?"

## Ask first

| Ask | Why |
|---|---|
| The cost of a false positive and of a false negative, even roughly (in money, time, user harm) | the threshold follows from their ratio |
| A capacity limit (reviewers handle 500 cases a day)? | then the threshold is set by k, not by cost |
| The base rate now, and will it change in production? | precision depends on it; a threshold tuned at 10 % positives misbehaves at 1 % |
| Are the scores calibrated probabilities? | a cost-based threshold `p > c_FP / (c_FP + c_FN)` needs calibrated scores |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the current threshold is too strict / too loose | the expected cost at the current threshold is within noise of the minimum |
| the scores are calibrated | the calibration curve departs from the diagonal |
| one threshold fits all segments | per-segment precision / recall at that threshold differ widely |
| the trade-off is stable over time | precision at the threshold drifts across months |

## The notebooks

1. **Score distribution per class**: `px.histogram(x="score", color="label", barmode="overlay", histnorm="probability density")`; the current threshold as a vertical line.
2. **Precision-recall curve**: `sklearn.metrics.precision_recall_curve(y, score)`, as `px.line(x="recall", y="precision")`, the current threshold marked; ROC as a second view.
3. **Confusion counts per threshold**: TP, FP, FN, TN over a grid of thresholds (one `pd.DataFrame`), so each point can be read in counts, not rates.
4. **Expected cost per threshold**: `cost = c_FP × FP + c_FN × FN` (per 1,000 cases), `px.line(x="threshold", y="cost")`, the minimum and the current threshold marked; also with the costs halved and doubled (the answer's sensitivity).
5. **Calibration**: `sklearn.calibration.calibration_curve`; if calibrated, check the minimum lands near `c_FP / (c_FP + c_FN)`.
6. **Capacity**: if reviewers can take k cases, the threshold is the k-th highest score; show the precision and recall it gives.
7. **Base-rate shift**: recompute precision at the chosen threshold with the expected production base rate (reweight the classes); `px.line` of precision vs base rate.
8. **Per segment and over time**: precision and recall at the chosen threshold per segment and per month (`px.bar`, `px.line`).

## The test

A CI on precision and recall at the chosen threshold (`tests.md`, Wilson interval on the counts); two thresholds compared on the same cases: McNemar on the cases they classify differently.

## Traps

| Trap | Fix |
|---|---|
| a threshold of 0.5 by default | the cost ratio, or the capacity |
| accuracy or F1 chosen when the costs are known and unequal | expected cost |
| tuned at the test set's base rate, deployed at another | reweight to the production base rate |
| uncalibrated scores read as probabilities | calibrate, or pick the threshold on the cost curve directly |
| one threshold for segments that behave differently | per-segment check; per-segment thresholds only if justified and allowed |

## The answer's shape

The recommended threshold, the precision, recall and expected cost it gives (with CIs) next to the current one's, how sensitive it is to the cost assumptions, and what to monitor after the change (precision over time, the base rate).
