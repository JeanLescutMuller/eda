# D. Which model, checkpoint or training mix is better?

Comparing trained models (checkpoints, data mixes, hyperparameters, architectures) on an evaluation set. Usually: several runs per variant (seeds), every run scored on the same eval items, each item scored one or more times (samples, pass@k).

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| variant B beats A | the paired per-item difference's 95 % CI includes 0 |
| the gap is seed noise | the gap is larger than the spread between seeds of one variant |
| the gap comes from contaminated items | the gap holds on the clean items alone |
| the gap is in one family or difficulty level only | the per-family differences all point the same way |
| the variant trained more on a family wins on that family | the per-family gain doesn't follow the training mix |
| a broken grader or item (always 0 % or 100 %) drives it | the gap holds without those items |

## The test

`tests.md`: paired t / Wilcoxon on per-item differences; McNemar on pass / fail per item when each item is scored once; a mixed model or a two-level bootstrap with seeds; Holm when several variants are each compared to a baseline.

## Traps

| Trap | Fix |
|---|---|
| samples treated as independent (4 samples × 400 items = "1,600 observations") | aggregate to the item first |
| per-item spread mistaken for noise on the variant's effect | the paired difference removes the items' spread |
| seed noise ignored | show every seed; with 3 seeds say how big a gap they could detect |
| an unpaired test on a paired design | `ttest_rel`, not `ttest_ind` |
| a variant that changes two things | say the gap can't be attributed to either |
| a contaminated eval | rerun on the clean items before any conclusion |
| a winner picked among many variants or segments | correct for the comparisons; the second best is often within noise |

## The answer's shape

The recommended variant, its gain over the baseline in points with a 95 % CI, on the clean items, per segment where it matters; and what would settle it if it's not conclusive (more seeds, more items, a cleaner eval), with the n needed (`tests.md`, power).
