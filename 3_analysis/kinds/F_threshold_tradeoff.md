# F. Where should the cut-off be?

Choosing a threshold on a score (fraud, spam, a quality filter, an alert, a model's confidence) so that the errors on each side cost the least, given what each kind of error costs and how often each class occurs.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the current threshold is too strict / too loose | the expected cost at the current threshold is within noise of the minimum |
| the scores are calibrated | the calibration curve departs from the diagonal |
| one threshold fits all segments | per-segment precision / recall at that threshold differ widely |
| the trade-off is stable over time | precision at the threshold drifts across months |

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
