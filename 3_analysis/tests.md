# Tests: which one, and what the number says

A practical lookup, not a statistics course. Four gates, in order, before any `scipy` call; then the test is a table lookup.

## The 4 gates

| Gate | Ask | If skipped |
|---|---|---|
| 0 | **Is this a hypothesis, or a bug?** If the mechanical cause is already known (a unit, a parser, a join, a version), state it, size it, stop | a p-value on a deterministic gap: meaningless |
| 1 | **What type is the outcome?** continuous (a measurement), binary (pass / fail, converted), count (events per unit), categorical (one of several labels) | the wrong family of test |
| 2 | **How many groups, and paired or independent?** Paired = the same unit measured under each condition (before / after, two models on the same items). And what is the unit: rows are rarely independent | a paired design tested as independent loses most of its power; rows tested as if independent give p-values far too small |
| 3 | **H0 in one sentence, and what would disprove it**, before computing anything | a test chosen to fit the result |

## Outcome × design → test

| Outcome | Design | Test | Call | p answers |
|---|---|---|---|---|
| continuous | 1 sample vs a known value | one-sample t | `scipy.stats.ttest_1samp(x, mu)` | does the mean differ from the benchmark |
| continuous | 2 independent groups | Welch's t | `ttest_ind(a, b, equal_var=False)` | do the means differ |
| continuous | 2 independent, skewed or small n | Mann-Whitney U | `mannwhitneyu(a, b)` | does one group tend higher |
| continuous | paired | paired t | `ttest_rel(a, b)` | is the within-unit difference non-zero |
| continuous | paired, skewed or small n | Wilcoxon signed-rank | `wilcoxon(a, b)` | same, no normality assumed |
| continuous | 3+ independent groups | one-way ANOVA | `f_oneway(*groups)` | does any group mean differ |
| continuous | 3+ groups, skewed | Kruskal-Wallis | `kruskal(*groups)` | same, on ranks |
| continuous | 3+ conditions, paired | repeated-measures ANOVA / Friedman | `statsmodels AnovaRM` / `friedmanchisquare` | does any condition differ, within units |
| continuous × continuous | association | Pearson / Spearman | `pearsonr(x, y)` / `spearmanr(x, y)` | linear / monotonic relationship |
| binary | 1 sample vs a known rate | binomial | `binomtest(k, n, p)` | does the rate differ from the benchmark |
| binary | 2 independent groups | two-proportion z | `statsmodels proportions_ztest(counts, nobs)` | do the rates differ |
| binary | small counts (< 5 per cell) | Fisher's exact | `fisher_exact(table)` | same, exact |
| binary | paired | McNemar | `statsmodels mcnemar(table)` | do the units flip more one way than the other |
| categorical × categorical | independence | chi-square | `chi2_contingency(table)` | are the two variables associated |
| count / rate | 2 rates per time or exposure | Poisson rate test | `statsmodels test_poisson_2indep(c1, n1, c2, n2)` | do the underlying rates differ |
| any statistic | no named test fits | bootstrap / permutation | `scipy.stats.bootstrap`, `permutation_test` | a CI or a p with no formula |
| outcome with several factors | adjusting for confounders | regression | `statsmodels.formula.api.ols` / `logit("y ~ group + segment", df)` | the group's effect, the segment held fixed |
| repeated units (items × runs, users × days) | nested noise | mixed model, or a cluster bootstrap | `statsmodels mixedlm("y ~ group", df, groups=df.item)` | the group's effect, with each unit's own noise |

## Business phrasing → H0

| You hear | H0 | Disproved if |
|---|---|---|
| "Is X different between A and B?" | X's mean / rate is the same in A and B | the gap is larger than chance produces |
| "Did the change help?" | the metric is the same before and after (or treatment = control) | the improvement exceeds the noise |
| "Is this rate too high?" | the rate equals the benchmark | the observed rate is far enough from it |
| "Are these related?" | the variables are independent | the association is stronger than chance |
| "Is this segment worse?" | the segment equals the rest | the gap exceeds sampling noise |
| "Is it a data bug?" | not a hypothesis: gate 0 | recompute the number directly |

## Confidence intervals

Always report the effect and its 95 % CI next to the p-value: the CI says how big, the p-value only says "not zero".

- A mean: `mean ± 1.96 × sem` (`df.groupby(g).y.agg(["mean", "sem"])`), shown with `px.bar(..., error_y=...)`.
- A proportion: `statsmodels.stats.proportion.proportion_confint(k, n, method="wilson")`.
- A difference of means or proportions: the CI of the difference, not two overlapping CIs (two CIs can overlap while the difference is significant).
- Anything else (a median, a ratio, a percentile): bootstrap, resampling **the unit of analysis** (items, users, runs), not the rows.

## Bootstrap and permutation

- **Bootstrap a CI**: resample the units with replacement ~10,000 times, recompute the statistic, take the 2.5th and 97.5th percentiles. `scipy.stats.bootstrap((x,), np.mean)`.
- **Permutation p-value**: shuffle the group labels ~10,000 times, recompute the difference, the p-value is the share at least as extreme as the real one. `scipy.stats.permutation_test`.
- **Nested units** (items scored by several runs): resample at the top level (items), keep each item's runs together; or resample both levels when both vary (items and seeds).

## Many comparisons

Testing k things at α = 0.05: about k / 20 come out "significant" by chance.

- A few planned comparisons: Bonferroni (`α / k`) or Holm (`statsmodels.stats.multitest.multipletests(p, method="holm")`).
- Many (every column, every segment): Benjamini-Hochberg (`method="fdr_bh"`), and treat a lone hit as a lead.
- Say the k: "1 significant among 18 segments" reads very differently from "significant".

## One-sided or two-sided

Two-sided by default. One-sided only when the direction was stated before looking at the data ("is B better?" decided in advance) and the other direction would lead to the same decision as no difference.

## Power and sample size

- Before a test (design): `statsmodels.stats.power.tt_ind_solve_power(effect_size=d, alpha=0.05, power=0.8)`, or `NormalIndPower` for proportions (`proportion_effectsize(p1, p2)`).
- After a "no difference": report the CI's width. A small sample can't rule out a large effect: "no significant difference, but the CI is −8 to +6 points" is the honest version.
- The minimum detectable effect for the n you have, when the human asks "can we even tell?".

## Traps

| Trap | Fix |
|---|---|
| testing a bug | gate 0 |
| confounded groups (A vs B also differ by version, date, source) | the "what changes with the groups" check of `1_table/table.md`; then compare within the confounder, or adjust with a regression |
| rows treated as independent (samples within items, events within users) | aggregate to the unit first, or a mixed model / cluster bootstrap |
| scanning many columns or segments | the corrections above |
| "no difference" from a tiny sample | report the CI |
| a p-value with no effect size | the effect and its CI, always |
| a p-value read as the probability H0 is true | it's the chance of a gap this large if H0 were true |
| one-sided chosen after seeing the result | two-sided |
