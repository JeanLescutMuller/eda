# C. Read an A/B test: did the change work?

A randomized experiment already ran: users (or sessions, devices, accounts) were split into control and one or more treatments, and an outcome was logged. The job is to say whether the treatment moved the metric, by how much, and whether the test itself can be trusted.

## Hypotheses to list

| Hypothesis | Disproved if |
|---|---|
| the split is broken (bug in assignment or logging) | the sample-ratio chi-square p > 0.001 |
| treatment moves the primary metric | the difference's 95 % CI includes 0 |
| the gain is a novelty effect | the daily treatment effect stays flat over the test's days |
| a guardrail degrades (latency, revenue, errors, unsubscribes) | each guardrail's CI excludes a harmful move |
| the effect is in one segment only (platform, country, new vs returning) | the per-segment effects point the same way (a lead only, corrected for k) |
| pre-period imbalance explains the gap | the pre-period metric is the same in both arms |

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
