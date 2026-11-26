# F. Where should the budget, the data or the effort go?

Splitting a fixed resource between options (marketing channels, regions, teams, data sources in a training mix, labelling budget) to maximise an outcome, given how each option's return changes as it gets more.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| returns diminish with spend in each option | the response curve is linear over the observed range |
| option A's marginal return is higher than B's at current spend | the curves' slopes at the current points are within each other's CI |
| the current split is near the optimum | the optimiser's split differs from it by more than the uncertainty |
| options interact (one channel feeds another; sources overlap) | the outcome is additive across options |

## The test

Whether two marginal returns differ: their bootstrap CIs; whether the new split beats the old: a holdout or geo test (`C_experiment_design.md`) before moving the whole budget.

## Traps

| Trap | Fix |
|---|---|
| average return used instead of marginal | response curves; compare slopes at the current spend |
| extrapolating a curve far beyond observed spend | cap the moves at the observed range; test beyond it |
| attributed, not incremental, returns | holdout or geo tests, or say the bias |
| interactions ignored (overlapping sources, channels feeding each other) | say so; test the joint change |
| a point optimum with no uncertainty | the bootstrap range |

## The answer's shape

The proposed split next to the current one, the expected gain with its range, the option with the highest marginal return now, the constraint that binds, and a test to confirm before the full move.
