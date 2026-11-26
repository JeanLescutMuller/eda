# G. Is the world the same? Distribution shift between two datasets

Two datasets that should look alike: train vs eval, last year vs this year, one source vs another, the training data vs production traffic. The job is to find what differs, how much, and whether it matters for the outcome.

## What you'll hear

- "Our model scores well offline but badly in production: why?"
- "Does the eval set look like what the model was trained on?"
- "The new data source is different from the old one: how?"
- "Is this month's traffic different from last month's?"

## Ask first

| Ask | Why |
|---|---|
| Which dataset is the reference, which the target? | shift is measured from one to the other |
| Which features does the model or the decision use? | a shift in an unused column matters less |
| Is the outcome (label) known in both? | label shift vs covariate shift need different fixes |
| Is a time ordering involved? | drift over time: plot per period, not two blocks |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a feature's distribution shifted (covariate shift) | its TVD / PSI / KS statistic is small |
| the label balance shifted (label shift) | the outcome rate is the same in both |
| the relationship between features and outcome changed (concept shift) | a model trained on one predicts the other equally well, per segment |
| the shift is one new category or source | the shift disappears without it |
| the shift explains the performance gap | reweighting the reference to the target closes the gap |

## The notebooks

1. **Per feature, side by side**: categories `px.histogram(x=col, color="dataset", barmode="group", histnorm="percent", text_auto=".0f")`; numbers `px.histogram(..., histnorm="probability density", barmode="overlay")` or `px.ecdf(x=col, color="dataset")`.
2. **One number per feature**, sorted bars (`px.bar(orientation="h")`), coloured by threshold:
   - categories: total variation distance `0.5 * abs(p - q).sum()`: ≥ 0.1 orange, ≥ 0.3 red;
   - numbers: KS statistic (`scipy.stats.ks_2samp`) or PSI on reference deciles (`sum((q - p) * log(q / p))`: < 0.1 stable, 0.1–0.25 orange, > 0.25 red). With large n every KS p-value is tiny: read the statistic, not the p.
3. **All features at once — the classifier two-sample test**: label rows by dataset, train a gradient-boosting classifier (`sklearn.ensemble.HistGradientBoostingClassifier`, `cross_val_score(scoring="roc_auc")`); AUC ≈ 0.5 no detectable shift, > 0.7 strong; its feature importances name what differs.
4. **Label shift**: the outcome rate per dataset with Wilson CIs; per segment.
5. **Concept shift**: the outcome rate per bin of the main feature, per dataset (`px.line(x="bin", y="rate", color="dataset")`): the curves should overlap.
6. **Does it matter?**: the performance per segment in the target vs the reference; importance-weight the reference by the classifier's odds `p / (1 - p)` and recompute the reference metric: if the gap closes, the shift explains it.
7. **Over time**: the shift statistic per month vs a fixed reference (`px.line`), to see a step (a pipeline change) vs a drift.

## The test

`tests.md`: chi-square for categories, KS for numbers (both over-powered at large n: report effect sizes); permutation test on the classifier's AUC; Holm across features if p-values are reported.

## Traps

| Trap | Fix |
|---|---|
| p-values on huge samples | TVD, PSI, KS statistic as effect sizes |
| one feature at a time misses joint shifts | the classifier two-sample test |
| a shift that doesn't affect the outcome | step 6: does it matter? |
| two blocks hiding a drift | the per-period plot |
| reweighting with extreme weights | clip the weights, report the effective sample size |

## The answer's shape

The features that shifted most, with their statistics; whether the shift is covariate, label or concept; whether it explains the performance gap; and the fix (reweight, resample, collect target-like data, retrain).
