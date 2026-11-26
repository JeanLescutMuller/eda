# B. Which groups are there?

Splitting users, customers, items or sessions into groups that behave differently, either along dimensions that already exist (country, plan, tenure) or with clusters found in the data. The goal is a few segments the business can act on, not an algorithm.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the population is not homogeneous on the outcome | the outcome's spread is the same within every candidate split |
| an existing dimension already separates the behaviour | its segments' outcome distributions overlap almost fully |
| a found cluster is real | it changes a lot with the seed, k or a resample |
| a cluster is only one feature's scale | it disappears once features are scaled |
| a segment is just size (big vs small users) | it holds after normalising by activity |

## The test

`tests.md`: Kruskal-Wallis on the outcome across segments, chi-square for a categorical outcome. Clusters are built to differ on their own features: a test on those features is circular; test only on an outcome not used to build them.

## Traps

| Trap | Fix |
|---|---|
| features on different scales | scale before clustering |
| skewed features (spend, counts) | log first |
| clusters that change with the seed | the stability check; report it |
| testing a cluster on the features it was built on | test on a held-out outcome |
| too many segments to act on | cap k at what the business can use |
| segments that are only activity level | normalise by activity, or say so |

## The answer's shape

3–6 named segments, each with its size, its profile in one line, its outcome, and the action it suggests; how stable they are; and whether an existing dimension would do almost as well (simpler to use).
