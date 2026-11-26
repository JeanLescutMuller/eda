# E. Who will churn, convert or buy?

Scoring each unit (user, customer, account, lead) by its chance of an event in a future window, so that someone can act on the top of the list.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the event is predictable at all | the model's PR-AUC is close to the base rate |
| recent activity drives it (recency, frequency) | a model on those alone is as good as the full one |
| the score is calibrated | the calibration curve departs from the diagonal |
| the top decile holds most of the events | the lift in the top decile is close to 1 |
| targeting the top scores changes outcomes | an uplift test (treated vs held-out within the top) shows no difference |

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
