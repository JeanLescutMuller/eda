# D. Which group is better: vendors, teams, regions, variants?

Two or more groups compared on an outcome, without a randomized split: vendors delivering data, teams resolving tickets, regions selling, suppliers' defect rates. The groups usually differ in more than the label.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| group A beats B | the difference's 95 % CI includes 0 |
| the gap is the mix of items (difficulty, category, size) | the gap holds within each segment, or after standardization |
| the gap is a different version, date or tool | the "what changes with the groups" check (`1_table/table.md`) shows no confounder |
| the gap is a few extreme units (one bad batch, one bad annotator) | the gap holds without the top contributors, or on medians |
| groups are measured differently (another grader, another definition) | the same grader or definition applies to all groups |

## The test

`tests.md`: Welch / two-proportion z for the raw gap; paired t or McNemar on shared items; regression for the adjusted gap; ANOVA / Kruskal-Wallis then Holm for 3+ groups; clustered errors or a cluster bootstrap when sub-units repeat.

## Traps

| Trap | Fix |
|---|---|
| different item mixes | stratify, standardize or adjust |
| a confounder that splits like the groups (version, date) | the TVD check; compare within the confounder |
| rows not independent (many rows per annotator or store) | aggregate, or cluster the errors |
| one bad sub-unit generalized to the group | the concentration plot |
| the best of many groups declared the winner | the omnibus test, Holm, and the CI of the winner vs the second |
| different measurement per group | the same grader or definition, or say it can't be compared |

## The answer's shape

The adjusted gap with its CI, whether it holds in each main segment, the confounder or concentration found if any, and the decision it supports (switch, keep, audit a sub-unit).
