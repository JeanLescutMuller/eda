# C. Read an A/B test: did the change work?

A randomized experiment already ran: users (or sessions, devices, accounts) were split into control and one or more treatments, and an outcome was logged. The job is to say whether the treatment moved the metric, by how much, and whether the test itself can be trusted.

## What you'll hear

- "Here's the data from last week's experiment: should we ship?"
- "Treatment shows +2 % conversion: is it real?"
- "The primary metric went up but revenue went down: what do we do?"
- "It was significant on day 3 and not anymore: what happened?"

## Ask first

| Ask | Why |
|---|---|
| What was randomized: users, sessions, devices, pages? | the analysis unit must be the randomization unit, or the variance is wrong |
| What was the planned split (50 / 50, 90 / 10) and duration? | the sample-ratio check needs the planned split; stopping early changes the reading |
| Which metric was declared primary, and which are guardrails? | one primary metric is tested; the others are guardrails or leads |
| Was the analysis plan (metric, duration, MDE) fixed before the start? | peeking or picking the metric after the fact inflates false positives |

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the split is broken (bug in assignment or logging) | the sample-ratio chi-square p > 0.001 |
| treatment moves the primary metric | the difference's 95 % CI includes 0 |
| the gain is a novelty effect | the daily treatment effect stays flat over the test's days |
| a guardrail degrades (latency, revenue, errors, unsubscribes) | each guardrail's CI excludes a harmful move |
| the effect is in one segment only (platform, country, new vs returning) | the per-segment effects point the same way (a lead only, corrected for k) |
| pre-period imbalance explains the gap | the pre-period metric is the same in both arms |

## The notebooks

1. **Sample-ratio mismatch (SRM)**: units per arm vs the planned split, `scipy.stats.chisquare(counts, f_exp=planned_shares * counts.sum())`. p < 0.001 is red: stop, the readout is invalid until the assignment is explained. Split the SRM per day and per platform to find where it breaks.
2. **Aggregate to the randomization unit**: one row per user (`groupby(["arm", "user_id"]).agg(...)`), never one row per event or page view.
3. **The primary metric per arm**, with its 95 % CI: `px.bar(x="arm", y="mean", error_y="ci", text_auto=".2f")`.
4. **The difference and its CI**:
   - a per-user mean or rate: Welch's t / two-proportion z (`tests.md`);
   - a ratio metric whose denominator isn't the unit (clicks per page view, revenue per session, with users randomized): the **delta method** for the variance of the ratio, or a bootstrap over users; never a t-test on page-level rows.
5. **CUPED** (variance reduction): `y_adj = y - theta * (x - x.mean())`, with `x` the same metric in the pre-period and `theta = cov(y, x) / var(x)` pooled across arms; same point estimate, a narrower CI. Show both.
6. **Over time**: the cumulative and daily effect per day (`px.line(x="day", y="effect", error_y=...)`). A decaying effect is novelty; a growing one is learning. Don't use this plot to decide when to stop.
7. **Guardrails**: one row per guardrail, the effect and its CI (`px.bar(orientation="h", error_x=...)`), red if harmful beyond its tolerance.
8. **Segments**: the effect per segment with its CI, Holm or BH across segments (`tests.md`). A segment effect is a lead for a new test, not a reason to ship to that segment.

## The test

`tests.md`: Welch's t or two-proportion z on per-unit values; delta method or bootstrap for ratio metrics; regression `ols("y ~ arm + x_pre", df)` as the CUPED equivalent; Holm across guardrails and segments; sequential tests (always-valid p-values) only if the test was designed for peeking.

## Traps

| Trap | Fix |
|---|---|
| SRM ignored | check it first; a broken split invalidates every number after it |
| events treated as independent (page views of the same user) | aggregate to the randomization unit, or delta method |
| peeking: stopping the day it turns significant | read at the planned duration, or a sequential test planned in advance |
| many metrics, one "significant" | one primary metric; the others corrected or reported as leads |
| a significant segment shipped alone | rerun a test on that segment |
| novelty or a weekday effect | full weeks, the daily effect plot |
| significance without size | the effect and its CI against the MDE that matters |

## The answer's shape

Ship / don't ship / rerun: the primary effect in absolute and relative terms with its 95 % CI, the SRM and guardrails status, and the main caveat (novelty, a segment, a short duration).
