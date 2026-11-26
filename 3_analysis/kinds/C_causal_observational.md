# C. Did it work, without an experiment?

The change already happened, to some units or at some date, with no randomization. The job is to find a comparison that stands in for "what would have happened otherwise", and to say what that comparison assumes.

## What you'll hear

- "We launched in Germany in March: did sales go up because of it?"
- "Users who use feature X retain better: does X cause retention?"
- "Customers above 1,000 points get free shipping: does it make them buy more?"
- "We changed the price on a date: what did it do?"

## Ask first

| Ask | Why |
|---|---|
| Who got the change, and how was that decided? | the selection rule decides which method is valid |
| When did it start, and did anything else change then? | a concurrent change (a season, a campaign) is confounded with it |
| Is there a group that didn't get it, and data before the change? | diff-in-diff needs both |
| Is there a cut-off (a score, a date, a threshold) that decided who got it? | regression discontinuity |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the change caused the gain | the comparison group moved the same way |
| the treated would have improved anyway (selection) | matched or weighted untreated units show the same gain |
| a concurrent event explains it | the gain also appears in untreated units at the same date |
| the pre-trends differ | treated and control trends are parallel before the change |
| the effect is an artifact of the cut-off | units just above and just below the threshold look alike on every other variable |

## The notebooks

1. **The picture**: the outcome over time for treated vs untreated (`px.line(x="week", y="y", color="group")`), with the change date as a vertical line (`.add_vline`).
2. Then the method the selection rule allows:

| Method | When | How | Assumes |
|---|---|---|---|
| **Difference-in-differences** | a treated and an untreated group, before and after | `ols("y ~ treated * post", df)`, the interaction is the effect; clustered errors by unit (`cov_type="cluster"`) | parallel trends: check on the pre-period plot, and an event-study plot of the effect per week before and after |
| **Matching / propensity weighting** | treatment chosen on observed traits | `logit("treated ~ x1 + x2", df)` for the propensity, then inverse-probability weights or nearest-neighbour matching; check balance after (standardized mean difference < 0.1 per covariate, `px.bar`) | no unobserved confounder; overlap of propensities (`px.histogram(color="treated")`) |
| **Regression discontinuity** | a threshold on a score decides treatment | `ols("y ~ above + score_centered + above:score_centered", df[abs(score_centered) < h])`, a few bandwidths h; the scatter of y vs score with the cut-off | units can't sort themselves just above the cut-off: a histogram of the score shows no bunching |
| **Interrupted time series** | everyone treated at one date, long history | `ols("y ~ t + post + t_since", df)` with seasonality terms; or a forecast from before the date vs the actual after | no other change at that date; the pre-trend would have continued |
| **Synthetic control** | one treated unit (a country, a city), many untreated | a weighted mix of untreated units fitted on the pre-period; the gap after is the effect; placebo runs on each untreated unit | the mix that tracked before keeps tracking |

3. **Robustness**: a placebo date (a fake change date in the pre-period should show no effect), a placebo outcome (one the change can't affect), and the estimate under 2–3 specifications side by side.

## The test

`tests.md`: regression with clustered standard errors by unit; bootstrap over units for matching estimates; placebo distribution as a permutation test for synthetic control.

## Traps

| Trap | Fix |
|---|---|
| correlation read as causation (feature users retain better) | ask who chooses the feature; propensity or a natural experiment |
| non-parallel pre-trends | the event-study plot; a different control group |
| a concurrent change | a control group exposed to it too, or a placebo date |
| errors not clustered by unit | `cov_type="cluster"` |
| poor overlap in propensities | trim to the common support, say who's excluded |
| one specification reported | 2–3, and the placebo tests |

## The answer's shape

The effect with its CI, the method and the one assumption it rests on, the evidence for that assumption (the pre-trend plot, the balance table, the placebo), and how a test would settle it.
