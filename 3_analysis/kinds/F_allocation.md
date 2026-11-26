# F. Where should the budget, the data or the effort go?

Splitting a fixed resource between options (marketing channels, regions, teams, data sources in a training mix, labelling budget) to maximise an outcome, given how each option's return changes as it gets more.

## What you'll hear

- "How should we split next year's marketing budget across channels?"
- "Which data sources should get more weight in the training mix?"
- "We have 10,000 more labels to buy: where should they go?"
- "Should we move headcount from team A to team B?"

## Ask first

| Ask | Why |
|---|---|
| The outcome to maximise, and its unit (conversions, revenue, eval score) | everything is compared in that unit |
| The constraints: total budget, minimum / maximum per option, contracts | the feasible set |
| What data shows each option's return at different spend levels? | one spend level per option gives an average, not the marginal return |
| Are returns incremental (measured against a holdout) or attributed? | attributed returns double-count and overrate the last-touch channels |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| returns diminish with spend in each option | the response curve is linear over the observed range |
| option A's marginal return is higher than B's at current spend | the curves' slopes at the current points are within each other's CI |
| the current split is near the optimum | the optimiser's split differs from it by more than the uncertainty |
| options interact (one channel feeds another; sources overlap) | the outcome is additive across options |

## The notebooks

1. **Spend and outcome per option over time**: `px.line(color="option")`; the current split as a `px.pie` or bar.
2. **Response curves**: outcome vs spend per option (`px.scatter(trendline="lowess", facet_col="option")`); fit a concave curve, e.g. `outcome = a × log(1 + spend / b)` or a power law, with `scipy.optimize.curve_fit`.
3. **Marginal return**: the derivative of each curve at the current spend, as a sorted `px.bar`; the rule: move budget from the lowest marginal return to the highest until they're equal.
4. **Optimise under constraints**: `scipy.optimize.minimize(-total_outcome, x0, constraints=[{"type": "eq", "fun": lambda x: x.sum() - budget}], bounds=...)`; a linear version with `scipy.optimize.linprog` when returns are constant per unit.
5. **Uncertainty**: refit the curves on bootstrap resamples (`tests.md`), rerun the optimiser, show the range of each option's share (`px.box`).
6. **Data mixtures**: the same logic on training data: eval score vs a source's share across trained mixes (`D_model_eval.md` for the comparison of mixes), plus each source's quality and overlap with the eval (`G_contamination.md`, `G_distribution_shift.md`); a source whose gain is within seed noise gets no more weight.
7. **The proposal**: current vs proposed split, side by side (`px.bar(barmode="group")`), and the expected gain with its range.

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
