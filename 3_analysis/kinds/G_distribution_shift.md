# G. Is the world the same? Distribution shift between two datasets

Two datasets that should look alike: train vs eval, last year vs this year, one source vs another, the training data vs production traffic. The job is to find what differs, how much, and whether it matters for the outcome.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| a feature's distribution shifted (covariate shift) | its TVD / PSI / KS statistic is small |
| the label balance shifted (label shift) | the outcome rate is the same in both |
| the relationship between features and outcome changed (concept shift) | a model trained on one predicts the other equally well, per segment |
| the shift is one new category or source | the shift disappears without it |
| the shift explains the performance gap | reweighting the reference to the target closes the gap |

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
